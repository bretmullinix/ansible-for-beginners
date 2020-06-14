# Working with Ansible Playbooks

Last updated: 06.14.2020

## Purpose

The purpose of this document is to show how to work with Ansible playbooks.

## Prerequisites

Please read the instructions in the [readme.md](../t1-getting-started/readme.md)
to have an understanding of how to set up your Ansible environment
and run adhoc Ansible commands before continuing.

Please setup your environment if you don't have access to Ansible by
following the instructions [here](../t1-getting-started/readme.md).

### A Running VM

You can create your VM anyway you want.  

One option is to use Terraform to create an Amazon EC2 instance (VM).
I have instructions on setting up your environment in the
[terraform for beginners git repo](https://github.com/bretmullinix/terraform-for-beginners).
In the repo be sure to follow the instructions in the
[readme.md](https://github.com/bretmullinix/terraform-for-beginners/blob/master/readme.md)
and the instructions in the
[ about how to inject your private key into an ec2 instance](https://github.com/bretmullinix/terraform-for-beginners/tree/master/injecting-your-ssh-key-into-ec2-instance).

### Instructions

1. Open up a terminal.
1. mkdir getting-started-with-playbooks
1. cd getting-started-with-playbooks
1. Copy the content [here](../t1-getting-started) to the current directory.
1. Run `tree` in your terminal.  You should see the following output:

    ![tree output](../images/getting-started-with-playbooks-initial-dir-tree-structure.png)

    The output of the **tree** command is the following:
    
    - **ansible.cfg**: The ansible configuration file used to configure
    the location of your inventory and other preferences for your playbook(s)
    in the current directory and below.
    - **inventory**: The directory contains all the inventory files.
     The inventory files define the servers and/or groups
     that can be a target for your playbook(s)
    
1.  Run `cat ansible.cfg | grep -v '^$' | grep -v '^#'`.  You will get
the following output:

    ![ansible.cfg configuration](../images/getting-started-with-playbooks-ansible-cfg-contents.png)

    The active configuration is explained below:
    
      - **inventory** = Currently, the value specifies the inventory directory to
        find all the inventory files.  If you had more than one
        inventory file, you could add them to this directory and ansible
        would allow you to specify any server or group(s)
        listed in any of the files.
        
      - **remote_user** = The user on the remote machine you
        plan to login as using ssh.
        
      - **ask_user** = If set to true, before you run an ansible command(s),
        the ansible program will prompt you for a password.  Since we are going to
        be using a private key file, we won't need a password prompt on ssh
        login.
        
      - **ask_sudo_pass** = If set to true, before any privileged ansible
        command(s) can be run, the ansible program
        will prompt you for a password. If your user requires a
        password when running a **sudo** command, 
        you will need to set this to true.
        
      - **private_key_file** = The private key file that is used to login using
        ssh.
      
      - **become** = If set to true, the environment is run in privileged mode.
      
      - **become_method** = The method to become the privileged user.  In
      this case we are using the **sudo** method.
      
      - **become_user** = The privileged user to run as.  In this case
      we are using the **root** user.
      
      - **become_ask_pass** = If set to true, when running Ansible, the
      Ansible program will prompt you for the privileged user's password.  In our
      case the login user does not require a password to change to the
      privileged user.

1. Copy your **my_key** private key file for your vm to this directory.
1. Edit the **./inventory/my_first_inventory** file and change the ip to
correspond to the ip of your VM.
1. Create a file called **my-first-playbook.yml**
1. Add the following content to the file:

    ```yaml
   ---
   - name: My First Playbook
     hosts: all
     tasks:
     - name: Print the Running Users Id
       command:
         argv: id
     - name: Install the Apache HTTP Server
       yum:
         name: httpd
         state: present
   ```
   :warning: Spaces matter in playbooks.  All child tags in `yaml` must
   be indented by 2 spaces from the parent tag.
   
   The definition of the above **playbook**:
   
   - **My First Playbook** = The name of your playbook.
   
   - **hosts** = The server(s) or group(s) to be the target servers for the
   playbook.  In this case, the **all** group of server(s) are the target(s)
   of the playbook. The **all** group is defined in the
    **./inventory/my_first_inventory** file.
   
   - **tasks** = Marking the start of defining **tasks**.  A **task**
   is a definition of the name of the task (optional), the module to run,
   and the required arguments for the module.

   - **Print the Running Users Id** Task = The task runs 
   the **command** module with the argument **id**.
   The **command** module runs any arbitrary command(s) listed
   in the arguments section, in this case the **id** command.
   
   - **Install the Apache HTTP Server** Task = The task runs the
   **yum** module with the arguments **name:  httpd** and **state: present**.
   The **yum** module looks for the package listed under the **name**
   argument and installs the package if the package is not installed.
   Otherwise, the **yum** module does nothing to the target server(s).
   
1. Run ` ansible-playbook --syntax-check my-first-playbook.yaml` to make
sure your ansible playbook is syntactically correct.  You may add **-v**,
**-vv**, **-vvv**, or **-vvvv** to show more and more detailed output for
debugging. 

    :eyes: If you get an error, make sure your spacing is correct in
    the playbook.  All child tags must be indented by 2 spaces.

1. Run `ansible-playbook my-first-playbook.yaml` to run the playbook.  The
playbook will check the **id** of the logged in user on the server(s) and then
install **httpd** if it does not exist.  You may add **-v**, **-vv**, **-vvv**,
or **-vvvv** to show more and more detailed output for debugging.

All resulting playbook file is in the current directory.