# Step by Step CI - Create a Vagrant Windows base box
A summary of the steps taken from [Huestones - Creating a Windows 10 Base Box][winbox]

### Summary
This is compacted version of the blog post metioned above. It is there so that I have a quick and easy reference to creating windows base boxes.

### Requirements
- [Virtual Box][virtbox]
- [Vagrant][vag]
- Windows ISO


### Creating the Initial VM
- Disk Space: **VDI**, use a large disk size.
- Memory: **1024** or **2048**, can still be changed later in the vagrant   script
- CPU: **1**
- Addtionally: Disable any unnecessary hardware e.g. audio. Leave 2D/3D acceleration disabled.
- Sharing Settings: Enable bidirectional shared clipboard and drap-drop support.

### Install Windows on the VM
- Username and Password: vagrant
- Install VirtualBox Guest Addtionals
- Install .Net 3.5 (Control Panel > Programs and Features > Turn Windows..)
- Disable UAC (Control Panel > User Accounts > Change user account control settings)
- run in Admin CMD
  ```
  reg add   HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System /v EnableLUA /d 0 /t REG_DWORD /f /reg:64
  ```
- Enable WinRM from admin CMD by running each command
```
winrm quickconfig -q
winrm set winrm/config/winrs @{MaxMemoryPerShellMB="512"}
winrm set winrm/config @{MaxTimeoutms="1800000"}
winrm set winrm/config/service @{AllowUnencrypted="true"}
winrm set winrm/config/service/auth @{Basic="true"}
sc config WinRM start= auto
```
- Relax Powershell execution policy from Admin CMD
```
Set-ExecutionPolicy -ExecutionPolicy Unrestricted
```
- Enable remote connections to box (Control Panel > System > Advanced System Settings > remote > Allow remote connections, uncheck "Only allow...NLA")
- Turn off System Protection (Control Panel > System > Advanced System Settings)
- Adjust Performance Settings to disable all visual styling (Control Panel > System > Advanced System Settings > Performance Settings)
- Adjust Page file size to max 1024MB (Control Panel > System > Advanced System Settings > Performance Settings)
- Install all current Windows updates
- Set the update settings to " Notify to scheduled restart" (Settings > Update & Security > Windows Update > Advanced Options)
- Run in Admin CMD to clean up some Space
```
C:\Windows\System32\cleanmgr.exe /d c:
```
- Disconnect any shared drives and virtual drives
- Restart and check that all updates are installed

### Exporting the base box
- Create a default vagrant file for the base box

```
  # -*- mode: ruby -*-
  # vi: set ft=ruby :

  # All Vagrant configuration is done below. The "2" in Vagrant.configure
  # configures the configuration version (we support older styles for
    # backwards compatibility). Please don't change it unless you know what
    # you're doing.
    Vagrant.configure(2) do |config|
    config.vm.guest = :windows
    config.vm.communicator = "winrm"
    config.vm.boot_timeout = 600
    config.vm.graceful_halt_timeout = 600

    # Create a forwarded port mapping which allows access to a specific port
    # within the machine from a port on the host machine. In the example below,
    # accessing "localhost:8080" will access port 80 on the guest machine.
    # config.vm.network "forwarded_port", guest: 80, host: 8080
    config.vm.network :forwarded_port, guest: 3389, host: 3389
    config.vm.network :forwarded_port, guest: 5985, host: 5985, id: "winrm", auto_correct: true

    # config.vm.provider "virtualbox" do |vb|
    #    # Customize the name of VM in VirtualBox manager UI:
    #    vb.name = "yourcompany-yourbox"
    # end
    end

```

- Export the box from Admin CMD
```
  vagrant package --base VirtualBoxVMName --output /path/to/output/windows.box --vagrantfile /path/to/initial/Vagrantfile
```

### Using the base box
- Start new project
- init the vagrant box with
```
vagrant box add /path/to/output/windows.box --name ANameForYourBox
```

### Resources and Links
http://huestones.co.uk/node/305 - The list above is a shortend version of this wonderful blog post. 

[winbox]: http://huestones.co.uk/node/305 "Huestones Win base box"
[virtbox]:  https://www.virtualbox.org/ "Oracle Virtual Box"
[vag]:      https://www.vagrantup.com/  "Vagrant"
[vag_base]: https://docs.vagrantup.com/v2/boxes/base.html "Vagrant base box"
