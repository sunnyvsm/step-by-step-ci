# Step by Step CI
A quick and dirty way to setup a CI envrionment on your local machine.

*Todo*
  - Add detailed descriptions of the thoughts of each step
  - Add a "how to" create a windows base box

### Summary
The idea is to create an easy way to spin up a CI, Testing a build environment to be able to quickly test concepts and ideas concerning the design of such infrastructure. In a second step, after the concept has been proven to be viable for the situation, the scripts should be able to provide close to the exact same environment except this time it should deliver a production ready system that can easily be destroyed and re-initialized reliably. This combination should also accommodate any changes that need to be made to the CI environment as life-cycle of the production continues.

*TL;DR*
- Allow quick iteration of the CI env. till proof of concept is accepted as good
- Allow for a reliable way to initialize, run and destroy the proven CI env. as the production CI.


**The Quick and Dirty**

*Note*: Installing the complete CI environment on your own dev box. This description is done on a windows 8 machine but should easily map to a OS X or Linux environment.

1. **Pre-requisits**
  1. Install on Dev box
    - [Oracle Virtual Box][virtbox]
    - [Vagrant][vag]
1. **Start up Jenkins Server**
  1. grab this repo
  1. In the Folder the following folder structure should exist
    ```
      CI
       | Jenkins VM
            | jenkins_data
    ```
  1. Open the command prompt in the Jenkins VM folder and run `vagrant up`
1. **Configure Jenkins Server**
  1. Goto http://localhost:8081 to access Jenkins
  1. Update all the currently available plugins
  1. Set the Jenkins URL to http://192.168.56.111:8080/
    - Manage Jenkins -> Configure System -> Jenkins Location


### References and Links
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
