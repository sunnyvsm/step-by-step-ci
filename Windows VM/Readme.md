# Step by Step CI - Add Windows Build Slave
A quick and dirty way to setup a CI envrionment on your local machine.

*Todo*
    - Add a "how to" create a windows base box

### Summary
This describes the minimum amount of steps needed to setup a Windows Build/Test Slave for the Jenkins master server created in [Jenkins Master Server](../Readme.md).


### Setup MS Build Slave
1. **Pre-requisits**
  1. Install on Dev box
    - [Oracle Virtual Box][virtbox]
    - [Vagrant][vag]
  2. Have a vagrant ready Windows base box named "win10"
1. Install [git-scm][git_scm] on the Windows VM
  - Important: When installing make sure to set the "Use Git from the Windows Command Prompt". If you do install git after installing the Slave Agent Service, remember to restart the service so that it reloads the command prompt commands it knows of.

1. ** Install Jenkins plugins**
  - [GitHub][jen_github]
  - [MSBuild][jen_msbuild]
  - [VSTestRunner][jen_vstest]
1. ** Configure Jenkins plugins**
  - Manage Jenkins -> Configure System
  - GitHub - nothing
  - MSBuild
    - Name: `.Net 4.0`
    - path to MSBuild: `C:\Windows\Microsoft.NET\Framework\v4.0.30319\MSBuild.exe`
    - Name: `.Net 3.5`
    - path to MSBuild: `C:\Windows\Microsoft.NET\Framework\v3.5\MSBuild.exe`
  - VSTest
    - Name: `VSTest Console`
    - Path to VSTest: `C:\Program Files (x86)\Microsoft Visual Studio 14.0\Common7\IDE\CommonExtensions\Microsoft\TestWindow\vstest.console.exe`
1. Setup Slave
  1. Install on the slave box
    - [Visual Studio][vs]
    - [Java][java]
1. Create Slave Node
  1. Manage Jenkins -> Manage Nodes
  1. New Node
    - Name: `winBuildSlave`
    - Type: `Dumb Slave`
  1. Confiuration
    - Remote root directory: `C:\`
    - Launch method: `Launch slave agents via Java Web Start`
    - Save
  1. Click on the new Node and follow one of the methods to install the slave-agent.jnlp on the slave machine. Remember to install the agent as a service after it has successfuly connected with the master server.



### References and Links

[virtbox]:  https://www.virtualbox.org/ "Oracle Virtual Box"
[vag]:      https://www.vagrantup.com/  "Vagrant"
[vs]:       https://www.visualstudio.com  "Visual Studio"
[java]:     https://www.java.com/en/    "Java"
[jen_github]:https://wiki.jenkins-ci.org/display/JENKINS/GitHub+Plugin
[jen_msbuild]:https://wiki.jenkins-ci.org/display/JENKINS/MSBuild+Plugin
[jen_vstest]:https://wiki.jenkins-ci.org/display/JENKINS/VsTestRunner+Plugin
[git_scm]:  https://git-scm.com/ "git scm"
