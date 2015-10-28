# Step by Step CI
A quick and dirty way to setup a CI envrionment on your local machine.

### Summary
The idea is to create an easy way to spin up a CI, Testing a build environment to be able to quickly test concepts and ideas concerning the design of such infrastructure. In a second step, after the concept has been proven to be viable for the situation, the scripts should be able to provide close to the exact same environment except this time it should deliver a production ready system that can easily be destroyed and re-initialized reliably. This combination should also accommodate any changes that need to be made to the CI environment as life-cycle of the production continues.

*TL;DR*
- Allow quick iteration of the CI env. till proof of concept is accepted as good
- Allow for a reliable way to initialize, run and destroy the proven CI env. as the production CI.


## The Quick and Dirty

*Note*: Installing the complete CI environment on your own dev box. This description is done on a windows 8 machine but should easily map to a OS X or Linux environment.

Pre-requisits
  1. Install on Dev box
    - [Oracle Virtual Box][virtbox]
    - [Vagrant][vag]

### Start up Jenkins Server
  1. grab this repo
  1. In the Folder the following folder structure should exist
    ```
      CI
       | Jenkins VM
            | jenkins_data
    ```
  1. Open the command prompt in the Jenkins VM folder and run `vagrant up`
      - *This will download and start up a linux box that is based on the [ubuntu/trusty64][atlas_ub] base box. This is the official base box from ubuntu and is downloaded from [Atlas][atlas] (HashiCorps online vagrant base box repository). After downloading the base box the script will install [Docker][docker]. After [Docker][docker] is installed, it will download and run the [offical docker jenkins image][docker_jen] from [Docker Hub][docker_hub].*
      - *Addtionally the vagrant file creates a shared folder(`jenkins_data`) between the VM and the host machine. This shared folder is passed through to the Docker image as the folder in which Jenkins will save its config information. This setup allows for a quick teardown and build up of the VM/Docker image without losing the Jenkins setup and installed plugins. This should also allow for a easy way to backup the Jenkins configuration and possibly even version control it.
     `Host Folder <-> VM <-> Docker Image`*
      - *Port forwarding is set to `Host 8081 -> VM 8080 -> Docker Image 8080`. This allows for Jenkins to be accessed from the host via `http://localhost:8081`*
      - *Addtional private network setup is done so that the Jenkins server will always have the same IP address and only be reachable via that IP address from within the Virtualbox network which all the slave nodes will be part of.*
      - *Lastly the vagrant user is given sudo rights to the docker command so that debugging the docker image via SSH is easier.*

### Configure Jenkins Server
  1. Goto http://localhost:8081 to access Jenkins
    - *This is done from the host machine and works because of the port forwarding done by the vagrant script.*
  1. Update all the currently available plugins
    - *`Managment Jenkins -> Manage Plugins`*
  1. Set the Jenkins URL to http://192.168.56.111:8080/
    - *`Manage Jenkins -> Configure System -> Jenkins Location`*

*All the relevant configuration files will be available in the VM shared folder (`jenkins_data`). The configuration will therefore be persisted even if the VM is destroyed.*

## Setup a Example Project
Once the Jenkis server is up and running and small example project can be created and built using a slave build node. A small test project can be used to test the CI chain to make sure it works before including a bigger project.

### Setup a simple Project that builds on a slave node
1. Setup a Windows Build Slave as described in [Windows Build Slave](/Windows VM/Readme.md)
1. Setup a Test Project in Jenkins (*The test project used in this example is: [OddState/MicroProjectForTesting
][oddstate_test]*)
  1. Create a new Item in Jenkins
    - Item name: `[anything you want to call it]`
    - Type: `Freestyle`
  1. In the more detailed Jenkins Project settings
    - GitHub project: `https://github.com/OddState/MicroProjectForTesting/`
    - check: `Restrict where this project can be run`
    - Label Expression: `winBuildSlave` (or whatever name you gave the windows build slave node.) *This is done to make sure that the ms build slave is used. Later, when there are more build slaves, this field can be set to a label that defines all ms build slaves so that the building/testing is done by a currently available node.*
    - Under *Source Code Managment*
      - Select: `Git`
      - repository URL: `https://github.com/OddState/MicroProjectForTesting.git`
    - Under *Build* Add a Build step
      - MSBuild Version: `.Net 4.0` *The choices here depend on what was configured for the MSBuild Plugin*
      - MSBuild Build File: `"MicroCalculator/MicroCalculator.sln"`
  1. Run the build.
      - Check the console output should the build fail to find out what caused the fail.

### Adding a step to run the VSTests
  1. Open the Jenkins Project configuration
  1. Add a build step `Run unit tests with VSTest.console`
    - VsTest Version: `VSTest Console`*The choices here depend on what was configured for the VSTest Runner Plugin*
    - Test Files: `"MicroCalculator/CalculatorFunctionTests/bin/Debug/CalculatorFunctionTests.dll"`
  1. Save and run the Jenkins Project.
    - *If it fails, check the console output, it can happen that the build fails because one of the unit tests failed. There is a unit test that randomly (~33%) fails. Just to keep things interesting. :P*

### Add a better way to see the failed tests
  1. Install the Jenkins [VSTest Plugin][jen_vstest].
    - *This will convert the VSTestRunner results to JUnit compatible results which can be read nicely by Jenkins.*
  1. Open the Project Configuration
  1. Add a Post-build Action `Publish MSTest test result report`
    - Test report TRX file: `${VSTEST_RESULT_TRX}`
  1. Save and run a build. Now there should be a `Test Result` link in the Build report.

### Adding Robotframework support
  1. In the test Project *[OddState/MicroProjectForTesting][oddstate_test]* there is a tiny test implmenetation of [Robotframework][roboframe] tests. These will now be added to the build project.
  1. Installing [Robotframework][roboframe] on the Windows Build Slave.
    1. Remote onto the build slave
    1. [*IronPython Robotframework install instructions*][ipy_robot]
      1. *Download IronPython and install it*
      1. *Add the path `C:\Program Files (x86)\IronPython 2.7` to the system `PATH` variable*
      1.  *Add the path `C:\Program Files (x86)\IronPython 2.7\Scripts` to the system `PATH` variable*
      1. *Download [elementree][ipy_el] zip file, extract it and run `ipy setup.py install` from within the extracted folder to install it.*
      1. *To Test if the framework is working correctly, open a command prompt in the `MicroCalculator\RobotTests` folder and run `ipybot operatorBehaviours.robot`. This should run a few tests and output some results.*
    1. Restart the Jenkins slave service
    1. In Jenkins install the [Robot Framework Plugin][jen_robo_plug]
    1. Open the Project Configuration and add a build step `Execute Windows batch command`
      - Command:
      ```
      ipybot -T -x xunit.xml -d RobotResults MicroCalculator\RobotTests\operatorBehaviours.robot
      exit 0
      ```
        - *T - adds timestam to the output files*
        - *x - adds a xunit compatible output xml*
        - *d - defines the output directory*

    1. Add a post-build action `Publish Robot Framework test results`
      - Directory of Robot output: `RobotResults`
      - Thresholds (both): `100`
    1. Save and run the project. In the build summary a Robot Test Summary should be visible.

### Compbining both test results into a single report
  1. Time to combine the 2 different test results into a single report.
  1. Install the Jenkins Plugin [Test Result Analyzer][jen_test_res]
  1. Because both test results are in xunit output format they are easily read by the plugin. No configuration is needed.
  1. Run the Project. Once its done the Report can be viewed by selecting it from the menu that is opened from next to the project name.
  1. Click `Get Build Report`


## References and Links
 - https://www.virtualbox.org/
 - https://www.vagrantup.com/
 - https://atlas.hashicorp.com/ubuntu/boxes/trusty64
 - https://wiki.jenkins-ci.org/display/JENKINS/GitHub+Plugin
 - https://wiki.jenkins-ci.org/display/JENKINS/MSBuild+Plugin
 -  https://wiki.jenkins-ci.org/display/JENKINS/VsTestRunner+Plugin
 - https://github.com/OddState/MicroProjectForTesting
 - https://github.com/OddState/MicroProjectForTesting.git
 - http://huestones.co.uk/node/305
 - https://tinkertry.com/how-to-change-windows-10-network-type-from-public-to-private
 - http://yakiloo.com/setup-jenkins-and-windows/
 - https://sites.google.com/site/sudokillall9/articles/jenkinsvisualstudio2013andcontinuousintegration


[virtbox]:  https://www.virtualbox.org/ "Oracle Virtual Box"
[vag]:      https://www.vagrantup.com/  "Vagrant"
[vs]:       https://www.visualstudio.com  "Visual Studio"
[java]:     https://www.java.com/en/    "Java"
[atlas]:    https://atlas.hashicorp.com/  "Atlas"
[atlas_ub]: https://atlas.hashicorp.com/ubuntu/boxes/trusty64 "Ubuntu"
[docker]:   https://www.docker.com/ "Docker"
[docker_jen]: https://hub.docker.com/_/jenkins/ "Offical Docker Jenkins Image"
[docker_hub]: https://hub.docker.com/ "Docker Hub"
[oddstate_test]:  https://github.com/OddState/MicroProjectForTesting "OddState - Test Project"
[jen_vstest]: https://wiki.jenkins-ci.org/display/JENKINS/MSTest+Plugin "VSTest Plugin"
[roboframe]: http://robotframework.org/ "Robotframework"
[ipy]:  http://ironpython.net/ "IronPython"
[ipy_robot]: https://github.com/robotframework/robotframework/blob/master/INSTALL.rst#ironpython-installation "Robotframework via IronPython"
[ipy_el]: http://effbot.org/downloads/#elementtree "Elementree"
[jen_robo_plug]: https://wiki.jenkins-ci.org/display/JENKINS/Robot+Framework+Plugin "Robot Framework Plugin"
[jen_test_res]: https://wiki.jenkins-ci.org/display/JENKINS/Test+Results+Analyzer+Plugin "Test Result Analyzer"
