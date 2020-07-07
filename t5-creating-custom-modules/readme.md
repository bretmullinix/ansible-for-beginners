# Creating Custom Ansible Modules

Last updated: 07.07.2020

## Purpose

The purpose of this document is to show how to create Ansible modules.
Sometimes a developer does not have a good Ansible module to leverage
for his/her purposes.  In this case, writing their own Ansible module might
make sense.

## Prerequisites

Please read the instructions in the [readme.md](../t1-getting-started/readme.md)
to have an understanding of how to set up your Ansible environment
and run adhoc Ansible commands before continuing.

Please setup your environment if you don't have access to Ansible by
following the instructions [here](../t1-getting-started/readme.md).

A working knowledge of Ansible playbooks.  If you haven't, please go
[here](../t2-using-playbooks) to learn more.

A working knowledge of Ansible variables.  If you haven't please go
[here](../t3-using-variables) to learn more.

A working knowledge of Ansible roles.  If you haven't please go
[here](../t4-organizing-your-ansible-code-with-roles) to learn more.

A working knowledge of **python**.

## Installation

### Install Python 3

1. Run `python --version`.  If the no version returns, or
you have a version less than 3.0, run the following steps.
Otherwise, you have Python 3 installed.
1. Run `yum install gcc openssl-devel bzip2-devel libffi-devel`
1. Run `cd /usr/src`
1. Run `wget https://www.python.org/ftp/python/3.7.7/Python-3.7.7.tgz`
1. Run `tar xzf Python-3.7.7.tgz`
1. Run `cd Python-3.7.7`
1. Run `./configure --enable-optimizations`
1. Run `make altinstall`
1. Run `rm /usr/src/Python-3.7.7.tgz` 

### Create a Python Virtual Environment

1. Open a terminal window
1. Run `mkdir virtual_environments`
1. Run `cd virtual_environments`
1. Run `python -m venv venv_custom_ansible_modules`
1. Run `source venv_custom_ansible_modules/bin/activate`
1. Run `pip list`.  You should only have a couple of items installed
like **pip** and **setuptools**
1. Lets upgrade **pip**.  Run `pip install --upgrade pip`
1. Run `pip list`.  Depending on your system, your version of **pip**
might have changed.
1. Let's install Ansible 2.9.  Run `pip install ansible==2.9`
1. Run `pip list`. You should see **ansible** with the version 2.9.
1. Let's install Molecule.  Molecule provides us the capabilities of
**Ansible Galaxy** and **Test Driven Design**.  Run `pip install molecule==3.0.4`
1. Let's Install Ruby needed for custom ansible module.  Run `pip install gem`
1. Let's Install **MarkDownLint**.  Run `gem install mdl`

## Instructions

1. Open up a terminal.
1. Navigate to your **virtual_environments** folder
1. Run `source venv_custom_ansible_modules/bin/activate`
1. Let's create an ansible role to keep all of our custom ansible modules.

    1. Run `mkdir -p ansible/roles`
    1. Run `cd ansible/roles`
    1. Run `molecule init role custom_ansible_modules`.  The previous command
    created the ansible module **custom_ansible_modules** just like **Ansible Galaxy**,
    except for a folder called **molecule**.  The
    **molecule** folder will be used to configure and run tests using
    **Test Driven Development**.

1. Run `cd custom_ansible_modules`
1. Run `mkdir library`.  This folder will contain all of our custom ansible
modules.
1. Run `cd library`
1. Create the file called **custom_markdown_lint.py**
1. Add the following content to the file:

    ```python
    #!/usr/bin/python3
    
    from ansible.module_utils.basic import AnsibleModule
    import re
    ANSIBLE_METADATA = {
        'metadata_version': '1.1',
        'status': ['preview'],
        'supported_by': 'community'
    }
    
    DOCUMENTATION = '''
            ---
            module: custom_markdown_lint
            
            short_description: Gets the list of Markdown Lint problems.
            
            version_added: "2.9"
            
            description:
                - "The module can report all the markdown lint problems for a file or folder."
            
            author:
                - Bret Mullinix
            '''
    
    EXAMPLES = '''
            # List all the markdown problems for test.md
            
            - name: List all the mark down problems for test.md
              custom_markdown_lint:
                name: test.md
                action: problems
            '''
    
    RETURN = '''
            markdown_lint_problems:
                description: All the markdown lint problems
                type: str
                returned: always
            '''
    
    
    def run_module():
        # define available arguments/parameters a user can pass to the module
        module_args = dict(
            name=dict(type='str', required=True),
            action=dict(type='str', required=True)
        )
    
    
        # seed the result dict in the object
        # we primarily care about changed and state
        # change is if this module effectively modified the target
        # state will include any data that you want your module to pass back
        # for consumption, for example, in a subsequent task
        result = dict(
            markdown_lint_problems='No problems'
        )
    
        # the AnsibleModule object will be our abstraction working with Ansible
        # this includes instantiation, a couple of common attr would be the
        # args/params passed to the execution, as well as if the module
        # supports check mode
        module = AnsibleModule(
            argument_spec=module_args,
            supports_check_mode=True
        )
    
        action_to_perform = module.params["action"]
        files_or_directories = module.params["name"]
        # if the user is working with this module in only check mode we do not
        # want to make any changes to the environment, just return the current
        # state with no modifications
        if module.check_mode:
            module.exit_json(**result)
    
        import subprocess
    
        command_to_run = ['mdl',files_or_directories]
    
        process = subprocess.Popen(command_to_run,
                                   stdout=subprocess.PIPE,
                                   stderr=subprocess.PIPE
                                   )
        stdout, stderr = process.communicate()
    
        raw_output = stdout.decode("utf-8").splitlines()
    
        result['markdown_lint_problems'] = raw_output
    
        # output
        no_failure = True
        if no_failure is False:
            failure_message = 'Failed.'
            module.fail_json(msg=failure_message, **result)
    
        # in the event of a successful module execution, you will want to
        # simple AnsibleModule.exit_json(), passing the key/value results
        module.exit_json(**result)
    
    
    def main():
        run_module()
    
    
    if __name__ == '__main__':
        main()

    ```
1. Run `cd ../../..`
1. Create the file called **playbook.yml**
1. Add the following to the file.

    ```yaml
    ---
    - name: Playbook to run a role
      hosts: localhost
      connection: local
      tasks:
        - name: Run the role
          include_role:
            name: custom_ansible_modules
   ```
1. Run `ansible-playbook -vvv playbook.yml`

    You should see the errors/warnings generated by mark down lint.