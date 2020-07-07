# Working with Ansible

Last updated: 07.07.2020

## Purpose

The purpose of the repo is to provide simple tutorials for Ansible.

## Tutorials

The tutorials in this repo are as follows:

1. **Getting Started**

    Shows the user how to set up their environment
    and run some Ansible.  The tutorial is located in the
    [**t1-getting-started**](./t1-getting-started) folder.

2. **Getting Started with Playbooks**

    Adds to tutorial 1 by introducing playbooks.
    The tutorial is located in the
    [**t2-using-playbooks**](./t2-using-playbooks) folder.

3. **Using Variables**

    Adds to tutorial 2 by showing how to work with variables in a playbook.
    The tutorial is located in the
    [**t3-using-variables**](./t3-using-variables) folder.

1. **Organizing Your Ansible Code With Roles**

    As you build for your Ansible project(s), you will recognize a proliferation
    of playbooks and variables, and soon, you will have a hard time recognizing 
    what your ansible code is doing.  To solve this problem, the playbooks
    can be refactored into Ansible roles.  The purpose of this tutorial is to
    show how to refactor your variables and playbooks into an Ansible role.    
    The tutorial is located in the
    [**t4-organizing-your-ansible-code-with-roles**](./t4-organizing-your-ansible-code-with-roles )
    folder.
    
1. **Creating Custom Ansible Modules**

    As you build your Ansible project(s), you will find a time when
     **ansible** might not be the best place to store your code.  This might
     be a time to create a custom ansible module.  The
     tutorial is located in the 
    [**t5-creating-custom-modules**](./t5-creating-custom-modules )
    folder.

## Future Reading

After you are finished with the tutorials above, try the following tutorials 
to improve your **ansible** skills:

1.  [Learning Ansible Molecule and Test Driven Design](https://github.com/bretmullinix/ansible-molecule-for-beginners)

1.  [Using Ansible and Ansible Molecule to Install OpenShift and IDM on AWS](https://github.com/bretmullinix/openshift-idm-cluster-on-aws)