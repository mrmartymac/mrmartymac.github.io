# Installing Scoop 7

The following is an installation guide for Scoop 7 on a new CentOS 7 Linux server.

For testing purposes, we have been using VirtualBox to create virtual machines.  You can download and install VirtualBox using this [link](https://www.virtualbox.org/). 

Creating a new machine is simple, there are a few details that should be followed and will be detailed at the end of this guide.

## Creating VirtualBox VM for Scoop 7

While normal production systems will be on dedicated hardware or virtual machines, VirtualBox is a great way to have a localized version of Scoop 7 for testing on any platform.  The following was documented using VirtualBox 5.2.12 on a Mac running macOS High Sierra 10.13.4.

We assume VirtualBox is installed, but in case you need it still here is the [link](https://www.virtualbox.org/wiki/Downloads) to the download pages for VirtualBox.

### Create new CentOS 7 VM

1. Open the VirtualBox Manager and click New

![][1]

[1]: images/scoop-7-installation/create-new-centos-7-vm.png

### Create Virtual Machine

Click Expert Mode.  This simplifies the creation into fewer wizard pages.

![][2]

[2]: images/scoop-7-installation/create-virtual-machine.png

### Create Virtual Machine

1. Enter a **Name** for the VM.
1. Set the **Type** to Linux.
1. Set **Version** to: **Other Linux (64-bit).**
1. For a test machine 2GB of memory is adequate.  Enter **2048 MB** in the **Memory size**.
1. Choose **Create a virtual hard disk now.**
1. Click **Create**.

![][3]

[3]: images/scoop-7-installation/create-virtual-machine-1.png

### Create Virtual Hard Disk

Unless you anticipate importing a large amount of data for this test machine, no changes should be needed on this dialog.  Simply click **Create**.  If you would like to make the virtual hard disk larger, simply increase the **File size**.

![][4]

[4]: images/scoop-7-installation/create-virtual-hard-disk.png

### VERY IMPORTANT!!!

Before going any further you must open **Settings** and configure the Network adapter.  If you miss this step it causes problems that are difficult to resolve and you should simply start over.

1. Select the VM you just created.
1. Click Settings.

![][5]

[5]: images/scoop-7-installation/very-important---.png

### Configuring the Network adapter

1. Click on the **Network** button at the top (Mac as shown) or on the left side on Windows.
1. Set **Attached to** to the **Bridged Adapter**.
1. Click **OK**.

![][6]

[6]: images/scoop-7-installation/configuring-the-network-adapter.png

## First Startup & OS installation

To this point you have have essentially built the computer, you now need to install and configure the Linux operating system.

You will need to download the CentOS 7 ISO to use during the next steps.  Here is a [link](https://www.centos.org/download/) to the CentOS download page.  

1. Please click the DVD ISO option to choose a mirror site.

![][7]

[7]: images/scoop-7-installation/first-startup--amp--os-installation.png

### CentOS Mirror

Any mirror is fine, just choose one.  Save the file in a location you will remember.  Once the download is complete, move on to the next step.

![][8]

[8]: images/scoop-7-installation/centos-mirror.png

### Start the VM

1. Click on the VM.
1. Click **Start.**

![][9]

[9]: images/scoop-7-installation/start-the-vm.png

### OS Installation

After clicking Start a terminal window will popup along with a dialog to aid you in loading the OS.  

1. Click the browse icon on the right side (1) and browse to the CentOS ISO you downloaded earlier.
1. Click Start.

![][10]

[10]: images/scoop-7-installation/os-installation.png

### Install CentOS 7

The installation has begun.  There is an autotimer that will automatically continue the installation or you may choose **Install CentOS 7 **and press enter to continue immediately.

![][11]

[11]: images/scoop-7-installation/install-centos-7.png

### Installer running

As the installation proceeds, you will see a variety of messages.

![][12]

[12]: images/scoop-7-installation/installer-running.png

### Setup wizard

Eventually a wizard will appear.

1. Choose your preferred language
1. Click Continue

![][13]

[13]: images/scoop-7-installation/setup-wizard.png

### Configure Network

1. Click on the Network & Host Name button.



![][14]

[14]: images/scoop-7-installation/configure-network.png

### Turn Ethernet ON

1. Click on the OFF/ON button to turn the **Ethernet** connection **ON.**
1. Write down the **IP Address** as you will need this later. 
1. Did you write down the IP address?
1. Click **Done**.

![][15]

[15]: images/scoop-7-installation/turn-ethernet-on.png

### Installation Destination

You may need to set the Installation Destination.  If you see the red message shown blow follow these instructions.

1. Click on Installation Destination button.

![][16]

[16]: images/scoop-7-installation/installation-destination.png

### Installation Destination

1. Click on the **ATA VBOX HARDDISK.**
1. Click **Done**.

![][17]

[17]: images/scoop-7-installation/installation-destination-1.png

### Ready to install

1. Click **Begin Installation**.

![][18]

[18]: images/scoop-7-installation/ready-to-install.png

### Setting passwords

1. Click on **Root Password.**

![][19]

[19]: images/scoop-7-installation/setting-passwords.png

### Setting root password

1. Enter a password
1. Confirm your password.
1. Click **Done**.

![][20]

[20]: images/scoop-7-installation/setting-root-password.png

### User Creation

It is up to you if you would like to create another account at this time, if so, click on the User Creation button and follow the same process as you just completed with the root user.

### Finish configuration

1. Click **Finish configuration**. (This may differ on some platforms)

![][21]

[21]: images/scoop-7-installation/finish-configuration.png

### Installation complete

At this point, CentOS is installed.  Please complete the process by clicking **Reboot**.

![][22]

[22]: images/scoop-7-installation/installation-complete.png

### Time to login

When the server is started a console window similar to the one below will open by default.  This login screen is not of much use, but you will have to leave it open for now.

![][23]

[23]: images/scoop-7-installation/time-to-login.png

### How to open an SSH session to the server

You are now working on what is effectively two computers.  Your Desktop computer and the VM you just finished creating.  In the next few steps you will be required to run commands on the VM you just created.  It is best to use copy/paste to do this as some of these commands are rather long .  The default VM window started when you start the Linux server does not support copy/paste so you need to open a terminal session and ssh to the server.  There are several options to accomplish this task, but built into each platform, Mac and Windows, is Command Line Interface (CLI) that can be used. 

Following is a guide to use these built-in interfaces.  If you understand what is being done here and have another method you prefer, please feel free to ssh to the VM and continue with the section titled **OS Configuration Changes**.

### Windows - Open PowerShell

You can use PowerShell to SSH to the VM. 

1. Press the **Window key** on your keyboard plus the letter **R**.
1. Type **powershell**
1. Press **Enter**



![][24]

[24]: images/scoop-7-installation/windows---open-powershell.png

### Mac - Open Terminal

On the Mac you can use Terminal to ssh to the VM.

1. Open **Finder**.
1. Choose **Applications - Utilities**.
1. Locate **Terminal **and open it by double clicking on Terminal.

![][25]

[25]: images/scoop-7-installation/mac---open-terminal.png

### Windows PowerShell

The result of running the **powershell** command will be a window similar to the one below.

![][26]

[26]: images/scoop-7-installation/windows-powershell.png

### Mac Terminal

The result of running the **Terminal** application will be a window similar to the one below.

![][27]

[27]: images/scoop-7-installation/mac-terminal.png

### Your first ssh connection

There is no difference from this point on how to open the ssh connection to the server so the steps below will work for both platforms.

Enter the command below and press enter

     ssh root@<TheIPAddressOfYourVM>

In place of <TheIPAddressOfYourVM> insert the IP address you noted earlier.  See the examples below

![][28]

[28]: images/scoop-7-installation/your-first-ssh-connection.png

### Your first ssh connection

The very first time you connect you will be asked to confirm that you want to accept the connection.  This will only happen the first time you connect to a new machine.

You must type **yes **and press enter.

![][29]

[29]: images/scoop-7-installation/your-first-ssh-connection-1.png

### Your first ssh connection

Enter the password you configured for the root user earlier.

![][30]

[30]: images/scoop-7-installation/your-first-ssh-connection-2.png

### Your first ssh connection

This is the screen that will result.  From this point on you can continue with the commands and instructions below.

![][31]

[31]: images/scoop-7-installation/your-first-ssh-connection-3.png

## OS Configuration Changes

There are a few OS level configuration changes to be made before we begin.  Using your PowerShell or Terminal session you will be able to easily complete the following steps.

## Disable the firewall

The firewall can be configured if you choose, but for the purposes of this guide we will be disabling the firewall

From the command prompt perform the following commands followed by pressing the **Enter** key.  

     systemctl stop firewalld.service

     systemctl disable firewalld.service

## Install Nano editor

In the terminal type the following to install Nano.  During the installation you will be prompted to enter "y" twice.

     yum install nano

This editor is simpler to use than vi.

## Disable SELinux

We will also be disabling Security-Enhanced Linux (SELinux).  You can read about SELinux [here](https://en.wikipedia.org/wiki/Security-Enhanced_Linux).

The following will allow you to edit the SELinux configuration file.

From the command prompt perform the following commands followed by pressing the **Enter** key.

     nano /etc/selinux/config

You will see something like the image below.

1. Move your cursor to the end of the line defining the SELINUX policy as highlighted below in red.
1. Backspace to remove **enforcing.**
1. Type **disabled** in place of enforcing.
1. Press CTRL+O to save
1. Press Enter as prompted to overwrixte the existing file
1. Press CTRL+X to exit nano 
1. Please reboot the server (not needed if updating the server)



     shutdown -r now





![][32]

[32]: images/scoop-7-installation/disable-selinux.png

## After editing

This is what the edit should look like after you finish

![][33]

[33]: images/scoop-7-installation/after-editing.png

## Reboot the server

After disabling SELinux, you need to reboot the server.  The command below will take restart the server for you.

     shutdown -r +0

## Copy install kit to server

Copy the TAR file from your Mac to the VM using scp.  

Open Terminal on the Mac

CD to the folder containing the TAR file

     scp <TAR File Name> <user>@<VM Name or IP>:/root

### Install Apache on the server

Enter the command:

     sudo yum install httpd

### Unzip and install

To continue with the installation you will need to list the current directory and take note of the full name of the **scoop.tar.gz** file.  There will be version numbers as part of the file name.  For example **"scoop-7.0.8-1.tar.gz". **

**NOTE:** *If this is the first time installing Scoop 7 you can use the Tab key to autocomplete the file and folder names.  Just type the first part of the file name, scoop in this case and press the Tab key.*

Untar the file

     tar xzvf scoop-<VERSION>.gz

Change directory to the folder created when you unzipped Scoop 7.  The folder will have the same name as the base name of the Scoop 7 zip archive.

     cd scoop-<VERSION>

Run the install script with the following command.

     ./install_scoop.sh

### File shares - Windows

The installation process will have created two default shares accessible to all users.  The ScoopImages share shown below is the location InDesign will look for images attached to articles.  The two shares are:

     \\<TheIPAddressOfYourVM>\ScoopImages
     \\<TheIPAddressOfYourVM>\PageTrack



### File shares - Mac

When prompted, choose Guest and click Connect.



NOTE: If there is a problem with permissions run the following command

     chmod -R ugo+w /srv/pagetrack

Then restart smb

     systemctl restart smb

![][34]

[34]: images/scoop-7-installation/file-shares---mac.png

### First run of Scoop Client

Upon the first startup, there is a default username and password of `admin` and `first_setup`. Start a client and login using these credentials.