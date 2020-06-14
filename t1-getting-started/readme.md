# Working with Ansible

Last updated: 06.11.2020

## Purpose

The purpose of this document is to show how to install and work with ansible.

## Prerequisites

### For Fedora users

Fedora 27 or newer.

### For Windows users

Windows 7 or newer.

### A Running VM

You can create your VM anyway you want.  

One option is to use Terraform to create an Amazon EC2 instance (VM).
I have instructions on setting up your environment in the
[terraform for beginners git repo](https://github.com/bretmullinix/terraform-for-beginners/t1-getting-started).
In the repo be sure to follow the instructions in the
[readme.md](https://github.com/bretmullinix/terraform-for-beginners/blob/master/t1-getting-started/readme.md)
and the instructions in the
[ about how to inject your private key into an ec2 instance](https://github.com/bretmullinix/terraform-for-beginners/tree/master/t1-injecting-your-ssh-key-into-ec2-instance).

### Installation

#### Installing Python 3 on Windows
1. Download the executable for Python 3.8.0
[here](https://www.python.org/ftp/python/3.8.0/python-3.8.0-amd64.exe).
1. Run the executable.
1. Open up a terminal
1. Type `python3.8 --version`
1. The output should show you are running Python 3.8.0.
1. If you see a different version, you might have to set the **PATH** variable

#### Installing Python 3 on Fedora
1. Open up a terminal
4. sudo dnf install python3.8
1. Type `python3.8 --version`
1. The output should show you are running Python 3.8.0

### Instructions

1. Open up a terminal
1. Navigate to a directory where you plan on putting your
python virtual environments.

    :warning: You must always work out of a virtual environment.
    Virtual environments prevent you from corrupting
    your default system virtual environment and allow users to install different
    software for each virtual environment.

1. Run `python3.8 -m venv venv_ansible`
1. To activate your virtual environment on **Windows**, you run
`./venv_ansible/Scripts/activate`
1. To activate your virtual environment on **Fedora**, you run
`source ./venv_ansible/bin/activate`
1. Run `python --version`.  This is the version of Python running in your
virtual environment.
1. Run `pip install --upgrade pip`
1. Run `pip list`.  This should list the modules currently installed in your
environment.  Notice how ansible is not present.
1. Run `pip install ansible==2.9`.  The command installs **ansible 2.9** in the
virtual environment
1. Run `pip list` to confirm **ansible 2.9** is installed.

Ansible requires an **inventory** file.  An **inventory** file is used to
list the servers that you intend to run your ansible commands against.
The servers are typically under a **group** or **groups**.  In our case,
we will be using the **all** group.

1. mkdir inventory
1. vi inventory/my_first_inventory
1. Copy the following content into the file:

    ```
    [all]
    my_vm ansible_host=192.168.10.15
    ```
   
1. Replace the ip address with your vm ip address.
1. Save the file.
1. vi ansible.cfg
1. Under the **[defaults]** section, add the following line:

    ```
    inventory = ./inventory
    remote_user = maintuser
    ask_pass = false 
    ask_sudo_pass = false
    private_key_file = ./my-key
    ```
   
    In the line above, we are doing the following:
    
    - **inventory** = specifying the inventory directory to
    find all the inventory files.  If you had more than one
    inventory file, you could add them to this directory and ansible
    would allow you to specify any server or group(s) listed in any of the files.
    - **remote_user** = the user on the remote machine you
    plan to login as using ssh.
    - **ask_user** = if set to true, before you run an ansible command(s),
    the ansible program will prompt you for a password.  Since we are going to
    be using a private key file, we won't need a password prompt on ssh
    login.
    - **ask_sudo_pass** = if set to true, before any privileged ansible
    command(s) can be run, the ansible program
    will prompt you for a password. If your user requires a
    password when running a **sudo** command, you will need to set this to true.
    - **private_key_file** = the private key file that is used to login using
    ssh.    
    
### Run an Ansible Command

The format to run an ansible command after creating the configuration above
is using the following syntax:

**ansible [group or server] -m [module to run] -a '[arguments for module]'**

Sections to replace above in order to run a command:

- **[group or server name]** = the group or server defined in one of your
inventory files.
- **[module to run]** = the ansible module to run
- **[arguments for module]** = the ansible module arguments

For example, to get the "id" of the user running the ansible commands
on the remote server(s) run the following:

**ansible all -m command -a 'id'**

The command above says to run against the **all** group of servers using
the **command** module with an argument of **id**.

All artifacts are located in the **getting-started** folder.

To continue learning about Ansible, take a look at
[getting started with playbook](../t2-using-playbooks).



