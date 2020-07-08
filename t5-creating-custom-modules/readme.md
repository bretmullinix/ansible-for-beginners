# Creating Custom Ansible Modules

Last updated: 07.07.2020

## Purpose

Sometimes a developer does not have a good ansible module to leverage
for his/her purposes.  In this case, writing their own ansible module might
make sense.

The purpose of this document is to show how to create Ansible modules.


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
1. Let's Install Ruby needed for a custom ansible module.  Run `pip install gem`
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
            
            - name: List all the mark down problems for test.md in the roles folder.
              custom_markdown_lint:
                name: {{role_path }}/test.md
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

1. Let's breakdown the custom ansible module:

    ```python
    from ansible.module_utils.basic import AnsibleModule
    ```
   
   The code above imports the minimum ansible
   library needed to create an ansible module in Python.
   
   ```python
    ANSIBLE_METADATA = {
           'metadata_version': '1.1',
           'status': ['preview'],
           'supported_by': 'community'
       }
   ```
   
   The code above provides metadata for other tools to use.
   Below is what is contained in the current metadata:
         
     1. **metadata_version** --> The tag describes the metadata format, 
     so a tool can consume it properly.
        
     1. **status** --> The tag lists the status of the module. Here are some
     example statuses:
     
         1. **preview** --> The code is changing, whether it is the
         public interface or the internal code.
         
         1. **stableinterface** --> The public interface is stable.  Every
         effort will be made to keep the public interface as it is.
         
         1. **deprecated** --> The module will be removed in a future release.
         
         1. **removed** --> The module is not to be used.  The module
         might contain documentation to indicate the replacement for the
         module.
     1. **supported_by** --> Who supports the module.

   ```python
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
   ```

    The code above has the following documentation:
    
    1. **module** --> The name of the module.
    1. **short_description** --> A short description of what the module does.
    1. **version_added** --> The ansible version the module was added in.
    1. **description** --> A more detailed description of what the module does.
    1. **author** --> The author of the module.
    
   ```python
    EXAMPLES = '''
           # List all the markdown problems for test.md
           
           - name: List all the mark down problems for test.md in the roles folder.
             custom_markdown_lint:
               name: {{role_path }}/test.md
               action: problems
           '''
   ```
   
   The code above provides examples of the usage of the ansible module in
   an ansible playbook.
   
   ```python
   RETURN = '''
           markdown_lint_problems:
               description: All the markdown lint problems
               type: str
               returned: always
           '''
   ```
   
   The code above tells the ansible developer what output to expect
   from the ansible module.  This output can be captured by
   registering a variable in the ansible task.
   
   ```python
   def run_module():
     pass    
   ```
   
   The **run_module** is the method that is called by **main** method.
   When an ansible module runs, the **main** method runs, and the
   **run_module** method gets called.
   
   ```python
    module_args = dict(
               name=dict(type='str', required=True),
               action=dict(type='str', required=True)
           )
   ```
   
   **module_args** are used to declare the ansible module
   task attributes that are used when creating an ansible
   task.  The **EXAMPLES** code section above shows the 
   attributes used in an ansible task.

      1. **name** --> A required string attribute.  In this module,
      the attribute defines the folder or files to be scanned
      by markdown lint.
      
      1. **action** --> A required string attribute.  In this module,
      the action to be performed by markdown lint.

    ```python
    result = dict(
        markdown_lint_problems='No problems'
    )
    ```
   
   The code above initializes the **result** variable as a dictionary of
   key value pairs.  The dictionary is the output of the ansible
   module further down in the code.  The **key** listed is the
   **markdown_lint_problems**.  The value will show the output of calling
   markdown lint on the files or folders identified in the ansible **name**
   module attribute.
   
   ```python
   module = AnsibleModule(
       argument_spec=module_args,
       supports_check_mode=True
   )
   ```
   
   The code above declares the ansible module object 
   populated with the initial module definition.
   
   ```python
   action_to_perform = module.params["action"]
   files_or_directories = module.params["name"]
   ```
   
   The code above initializes two variables:

      1. **action_to_perform** --> The ansible **action** attribute value
      defined by the ansible developer in the ansible task.
      1. **files_or_directories** --> The ansible **name** attribute value
      defined by the ansible developer in the ansible task.
      
    ```python
    if module.check_mode:
        module.exit_json(**result)
    ```
   
   The code above checks to see if the **ansible** developer called the
   **ansible** playbook in **checks mode**.  If it was called in **checks
   mode**, the module exits without calling the markdown lint command, 
   and the module passes the default **results** dictionary back
   to the developer.
    
    ```python
        import subprocess
    
        command_to_run = ['mdl',files_or_directories]
    
        process = subprocess.Popen(command_to_run,
                                   stdout=subprocess.PIPE,
                                   stderr=subprocess.PIPE
                                   )
        stdout, stderr = process.communicate()
    
        raw_output = stdout.decode("utf-8").splitlines()
   ```
   
   The code above does the following:
       
    1.  **import subprocess** -->  The line of code imports the Python **subprocess**
    library.  The **subprocess** library calls shell commands on the target server(s).
    
    1.  **command_to_run = ['mdl',files_or_directories]** --> A list with the following:
    
        1. **mdl** --> The markdown lint command to run on the target server(s).
        1. **files_or_directories** --> The file or directories to run the
        **mdl** command on.  The variable was declared above and is assigned
        the value of the **name** attribute from the ansible task.
      
    ```python
    no_failure = True
    if no_failure is False:
        failure_message = 'Failed.'
        module.fail_json(msg=failure_message, **result)
    ```  
   The code above executes if a failure occurs.  In the current implementation,
   a failure never occurs because the variable **no_failure** is equal to **true**.
   Later we will refactor the code and add failure conditions.
   
   **module.fail_json(msg=failure_message, \*\*result)** --> The code
   informs ansible that the task has failed, and the code returns a 
   failure message and the results. 
  
    ```python
    module.exit_json(**result)
   ```

   The code above informs ansible that the **ansible** task completed successfully,
   and the code returns the results to the **ansible** playbook.
   
:construction: