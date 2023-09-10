# Condition

- Running a task with a condition `when`

## When

- `when` is not part of modules properties
- It must be indented at the same level as the module itself
- Example:

```yaml
---
- name: conditional install
  hosts: hap
  tasks:
  - name: Install apache on Centos
    yum:
      name: httpd
      state: latest
    when: ansible_facts['os_family'] == "RedHat"
  
  - name: Install apache on Ubuntu and Family
    apt:
      name: apache2
      state: latest 
    when: ansible_facts['os_family'] == "Debian"
```

- The variable that evaluate is not placed between double curly braces
- Notice when you `when`, the actual string should be double quote
- Execute the playbook using `ansible-playbook learning-5/playbooks/conInstall.yml --ask-pass`

```bash
kamal@TS-Kamal:~/github/ansible$ ansible-playbook learning-5/playbooks/conInstall.yml --ask-pass
SSH password: 
BECOME password[defaults to SSH password]: 

PLAY [conditional install] ***************************************************************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************************************************************
ok: [hap01.lab.rumah.lan]
ok: [hap02.lab.rumah.lan]

TASK [Install apache on Centos] **********************************************************************************************************************************************************************
changed: [hap02.lab.rumah.lan]
changed: [hap01.lab.rumah.lan]

TASK [Install apache on Ubuntu and Family] ***********************************************************************************************************************************************************
skipping: [hap01.lab.rumah.lan]
skipping: [hap02.lab.rumah.lan]

PLAY RECAP *******************************************************************************************************************************************************************************************
hap01.lab.rumah.lan        : ok=2    changed=1    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
hap02.lab.rumah.lan        : ok=2    changed=1    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   

kamal@TS-Kamal:~/github/ansible$ 
```

## Conditional Test Statements

- You also can use other conditional test statement.
- Example of conditional test statement:
  - Variable existence `is defined`
  - Variable not exist `is not defined`
  - Variable is true, 1 or yes - boolean `variable`
  - Variable is false, 0 or no - boolean `not variable`
  - Equal (string) `key == "value"`
  - Equal (numerical) `key == value`
  - Less then `key < value`
  - Less than or equal to `key <= value`
  - Not equal to `key != value`
- When using the `when` statement, the curly brackeyts doesn't needed because items in a `when` statement are considered to be variable by default.
- So, write like this `when: text == "TEST"` instead of `when: "{{ when }}" == "TEST"`
- Example - check existence:

```yaml
---
- name: check for existence of devices
  hosts: all
  tasks:
    - name: check if /dev/sda exists
      debug:
        msg: a disk device /dev/sda exists
      when: ansible_facts['devices']['sda'] is defined
```

- Example - check occurence of variable in a list:

```yaml
---
- name: test if variable is in another variables list
  hosts: hap
  vars_prompt:
  - name: my_answer
    prompt: which package do you want to install
  vars:
    supported_packages:
      - httpd
      - keepalived
  tasks:
   - name: Check requested packages allowed
     debug:
       msg: you are trying to install a supported package
     when: my_answer in supported_packages
```

- Example - Test multiple condition:
- Testing our multiple condition, you can use `and` or `or`

```yaml
---
- name: Check distribution and it release
  hosts: hap
  tasks:
  - name: showing output
    debug:
      msg: using CentOS 8
    when: ansible_facts['distribution_version'] == "8" and ansible_facts['distribution'] == "CentOS"
```

- Example - combining loop and condition:
- The variable/facts `['mounts']` got many mountpoint
- So you loop it and catches `/boot` then you compare and if it is larger than 100MB

```yaml
---
- name: conditionals test
  hosts: hap
  tasks:
  - name: Update kernal if sufficient space is available in /boot more than 100MB
    package:
      name: kernel
      state: latest
    loop: "{{ ansible_facts['mounts'] }}"
    when: item.mount == "/boot" and item.size_available > 100000000
```

## Handlers

- Handlers is like exception
- Let say if the 1st tasks failed then you don't want to run the handler but proceed with 3rd task
- You can use handler at the level you want - `notify` statement
- `notify` statement should list the name of the handler that is to be executed, and the handler listed at the end of the play
- Handle only trigger if the task triggered a **changed** status
- Exmple below - if the `copy nothing - intended to fail` failed then the handler - `restart_web` will skipped
- The `copy index.html` task must trigger a **changed** status to kick in the handler `restart_web`

```yaml
---
- name: create file on localhost
  hosts: localhost
  tasks:
  - name: create index.html on localhost
    copy:
      content: "welcome to the webserver"
      dest: /tmp/index.html

- name: set up web server
  hosts: hap
  tasks:
  - name: install httpd
    yum:
      name: httpd
      state: latest
  - name: copy index.html
    copy:
      src: /tmp/index.html
      dest: /var/www/html/index.html
    notify:
      - restart_web
  - name: copy nothing - intended to fail
    copy:
      src: /tmp/nothing
      dest: /var/www/html/nothing.html
  handlers:
    - name: restart_web
      service:
        name: httpd
        state: restarted
```

## Failures

### Tasks Errors

- There are three different types of result that generated by tasks:
  - `ok` - task has run succesfully but no changes were applied
  - `changed` - task has run successfully and changes have been applied
  - `failed` - While running the task, a failure condition was encountered.
- By default, if a task fails, all execution stops
- You can bypass using `ignore_errors` or `force handlers`
- If use `ignore_errors: yes`, the playbook continue, even after processing the fialing task
- If use `force_handlers`, if failed the handlers will continue to run. Even if a failing task was encounter
- You can use `failed_when` you want to specify task failure - ansible consider that a task success eventhough it is a failed command
- Example, below `echo hello world` and will stop if the `world` string exist

```bash
kamal@TS-Kamal:~/github/ansible$ ansible-playbook learning-5/playbooks/failedWhen.yml --ask-pass 
SSH password: 
BECOME password[defaults to SSH password]: 

PLAY [demonstrating failed_when] ***********************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************
ok: [hap01.lab.rumah.lan]
ok: [hap02.lab.rumah.lan]

TASK [run a script] ************************************************************************************************************************************************************************
fatal: [hap02.lab.rumah.lan]: FAILED! => {"changed": true, "cmd": ["echo", "hello", "world"], "delta": "0:00:00.001662", "end": "2023-09-10 16:37:25.750929", "failed_when_result": true, "msg": "", "rc": 0, "start": "2023-09-10 16:37:25.749267", "stderr": "", "stderr_lines": [], "stdout": "hello world", "stdout_lines": ["hello world"]}
...ignoring
fatal: [hap01.lab.rumah.lan]: FAILED! => {"changed": true, "cmd": ["echo", "hello", "world"], "delta": "0:00:00.001598", "end": "2023-09-10 16:37:26.338807", "failed_when_result": true, "msg": "", "rc": 0, "start": "2023-09-10 16:37:26.337209", "stderr": "", "stderr_lines": [], "stdout": "hello world", "stdout_lines": ["hello world"]}
...ignoring

TASK [see if we get here] ******************************************************************************************************************************************************************
ok: [hap01.lab.rumah.lan] => {
    "msg": "second task executed"
}
ok: [hap02.lab.rumah.lan] => {
    "msg": "second task executed"
}

PLAY RECAP *********************************************************************************************************************************************************************************
hap01.lab.rumah.lan        : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=1   
hap02.lab.rumah.lan        : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=1   

kamal@TS-Kamal:~/github/ansible$ 
```

- The sample a bit odd, alternatively we can use `fail` module.
- Execute the playbook using `ansible-playbook learning-5/playbooks/failedWhenFail.yml --ask-pass`
- In a use case where you want to stop the remaining tasks, you have to remove the `ignore_errors: yes` from the playbook header.

```bash
kamal@TS-Kamal:~/github/ansible$ ansible-playbook learning-5/playbooks/failedWhenFail.yml --ask-pass 
SSH password: 
BECOME password[defaults to SSH password]: 

PLAY [demonstrating the fail module] *******************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************
ok: [hap02.lab.rumah.lan]
ok: [hap01.lab.rumah.lan]

TASK [run a script] ************************************************************************************************************************************************************************
changed: [hap01.lab.rumah.lan]
changed: [hap02.lab.rumah.lan]

TASK [report a failure] ********************************************************************************************************************************************************************
fatal: [hap01.lab.rumah.lan]: FAILED! => {"changed": false, "msg": "the command has failed"}
...ignoring
fatal: [hap02.lab.rumah.lan]: FAILED! => {"changed": false, "msg": "the command has failed"}
...ignoring

TASK [see if we get here] ******************************************************************************************************************************************************************
ok: [hap01.lab.rumah.lan] => {
    "msg": "second task executed"
}
ok: [hap02.lab.rumah.lan] => {
    "msg": "second task executed"
}

PLAY RECAP *********************************************************************************************************************************************************************************
hap01.lab.rumah.lan        : ok=4    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=1   
hap02.lab.rumah.lan        : ok=4    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=1   
```

## Manipulate change status

- In ansible, the `changed` status are those change something or it also not changing something on the mange hosts
- Example, from the previous playbook. The command `echo` doesn't changed anything but the status state's `changed=1`
- At some point, you may need to manipulate the status so that it doesn't introduce confusion
- You can use `changed_when` in the task
- Example below disable `changed` status because it only run a `date` command
- Execute the playbook using `ansible-playbook learning-5/playbooks/disableChangeStatus.yml --ask-pass`

```bash
kamal@TS-Kamal:~/github/ansible$ ansible-playbook learning-5/playbooks/disableChangeStatus.yml --ask-pass 
SSH password: 
BECOME password[defaults to SSH password]: 

PLAY [demonstrate changed status] **********************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************
ok: [hap01.lab.rumah.lan]
ok: [hap02.lab.rumah.lan]

TASK [check local time] ********************************************************************************************************************************************************************
ok: [hap01.lab.rumah.lan]
ok: [hap02.lab.rumah.lan]

TASK [print local time] ********************************************************************************************************************************************************************
ok: [hap01.lab.rumah.lan] => {
    "command_result.stdout": "Sun 10 Sep 16:55:39 +08 2023"
}
ok: [hap02.lab.rumah.lan] => {
    "command_result.stdout": "Sun 10 Sep 16:55:38 +08 2023"
}

PLAY RECAP *********************************************************************************************************************************************************************************
hap01.lab.rumah.lan        : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
hap02.lab.rumah.lan        : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

kamal@TS-Kamal:~/github/ansible$ 
```

## Block

- Use when you have a group of tasks to tie with `when` statement
- In other word, you want to use condition for multiple task
- You specify the `block:` statement in the top most task and nested the child task

```yaml
- name: simple block example
  hosts: hap
  tasks:
   - name: setting up http
     block:
       - name: installing http
         yum:
           name: httpd
           state: present
       - name: restart httpd
         service:
           name: httpd
           state: started
     when: ansible_distribution == "Debian"
```

- Execute the playbook using `ansible-playbook learning-5/playbooks/block.yml --ask-pass`
- The task will skipped as be running Centos, both tasks were not executed

```bash
kamal@TS-Kamal:~/github/ansible$ ansible-playbook learning-5/playbooks/block.yml --ask-pass
SSH password: 
BECOME password[defaults to SSH password]: 

PLAY [simple block example] ****************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************
ok: [hap01.lab.rumah.lan]
ok: [hap02.lab.rumah.lan]

TASK [installing http] *********************************************************************************************************************************************************************
skipping: [hap01.lab.rumah.lan]
skipping: [hap02.lab.rumah.lan]

TASK [restart httpd] ***********************************************************************************************************************************************************************
skipping: [hap01.lab.rumah.lan]
skipping: [hap02.lab.rumah.lan]

PLAY RECAP *********************************************************************************************************************************************************************************
hap01.lab.rumah.lan        : ok=1    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
hap02.lab.rumah.lan        : ok=1    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   

kamal@TS-Kamal:~/github/ansible$ 
```

### Block rescue and always

- In the block statement, you can also put some exception handling
- You can use`rescue` where the following task will not be executed. However, when the a task success the `rescue` block will not be executed
- You can use `always` where the following task will be run regardless success or failure
- Example:

```yaml
---
- name: using blocks
  hosts: hap 
  tasks:
  - name: intended to be successful
    block:
    - name: remove a file
      shell:
        cmd: rm /var/www/html/index.html
    - name: printing status
      debug:
        msg: block task was operated
    rescue:
    - name: create a file
      shell:
        cmd: touch /tmp/rescuefile
    - name: printing rescue status
      debug:
        msg: rescue task was operated
    always:
     - name: always write a message to logs
       shell:
         cmd: logger hello
     - name: always printing this message
       debug:
         msg: this message is always printed
```

- Execute the playbook using `ansible-playbook learning-5/playbooks/rescueAlways.yml --ask-pass`

```bash
kamal@TS-Kamal:~/github/ansible$ ansible-playbook learning-5/playbooks/rescueAlways.yml --ask-pass
SSH password: 
BECOME password[defaults to SSH password]: 

PLAY [using blocks] ************************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************
ok: [hap01.lab.rumah.lan]
ok: [hap02.lab.rumah.lan]

TASK [remove a file] ***********************************************************************************************************************************************************************
fatal: [hap02.lab.rumah.lan]: FAILED! => {"changed": true, "cmd": "rm /var/www/html/index.html", "delta": "0:00:00.002606", "end": "2023-09-10 17:15:28.809127", "msg": "non-zero return code", "rc": 1, "start": "2023-09-10 17:15:28.806521", "stderr": "rm: cannot remove '/var/www/html/index.html': No such file or directory", "stderr_lines": ["rm: cannot remove '/var/www/html/index.html': No such file or directory"], "stdout": "", "stdout_lines": []}
fatal: [hap01.lab.rumah.lan]: FAILED! => {"changed": true, "cmd": "rm /var/www/html/index.html", "delta": "0:00:00.002551", "end": "2023-09-10 17:15:29.395982", "msg": "non-zero return code", "rc": 1, "start": "2023-09-10 17:15:29.393431", "stderr": "rm: cannot remove '/var/www/html/index.html': No such file or directory", "stderr_lines": ["rm: cannot remove '/var/www/html/index.html': No such file or directory"], "stdout": "", "stdout_lines": []}

TASK [create a file] ***********************************************************************************************************************************************************************
changed: [hap01.lab.rumah.lan]
changed: [hap02.lab.rumah.lan]

TASK [printing rescue status] **************************************************************************************************************************************************************
ok: [hap01.lab.rumah.lan] => {
    "msg": "rescue task was operated"
}
ok: [hap02.lab.rumah.lan] => {
    "msg": "rescue task was operated"
}

TASK [always write a message to logs] ******************************************************************************************************************************************************
changed: [hap01.lab.rumah.lan]
changed: [hap02.lab.rumah.lan]

TASK [always printing this message] ********************************************************************************************************************************************************
ok: [hap01.lab.rumah.lan] => {
    "msg": "this message is always printed"
}
ok: [hap02.lab.rumah.lan] => {
    "msg": "this message is always printed"
}

PLAY RECAP *********************************************************************************************************************************************************************************
hap01.lab.rumah.lan        : ok=5    changed=2    unreachable=0    failed=0    skipped=0    rescued=1    ignored=0   
hap02.lab.rumah.lan        : ok=5    changed=2    unreachable=0    failed=0    skipped=0    rescued=1    ignored=0   

kamal@TS-Kamal:~/github/ansible$ 
```

- You can see that the `rescue` statement executed and the `always` statement also executed
- At the time of execution, there were not directory for webservice
- We can remove the `always` statement to see whether the message log task run or not
- Execute the playbook using `ansible-playbook learning-5/playbooks/rescueOnly.yml --ask-pass`
- The remaining task will be executed as the part of the `rescue` statement. Unless the removing filed is success. All of the remaining task will be skipped

```bash
kamal@TS-Kamal:~/github/ansible$ ansible-playbook learning-5/playbooks/rescueOnly.yml --ask-pass
SSH password: 
BECOME password[defaults to SSH password]: 

PLAY [using blocks] ************************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************
ok: [hap02.lab.rumah.lan]
ok: [hap01.lab.rumah.lan]

TASK [remove a file] ***********************************************************************************************************************************************************************
fatal: [hap01.lab.rumah.lan]: FAILED! => {"changed": true, "cmd": "rm /var/www/html/index.html", "delta": "0:00:00.002248", "end": "2023-09-10 17:20:35.369419", "msg": "non-zero return code", "rc": 1, "start": "2023-09-10 17:20:35.367171", "stderr": "rm: cannot remove '/var/www/html/index.html': No such file or directory", "stderr_lines": ["rm: cannot remove '/var/www/html/index.html': No such file or directory"], "stdout": "", "stdout_lines": []}
fatal: [hap02.lab.rumah.lan]: FAILED! => {"changed": true, "cmd": "rm /var/www/html/index.html", "delta": "0:00:00.002212", "end": "2023-09-10 17:20:34.781638", "msg": "non-zero return code", "rc": 1, "start": "2023-09-10 17:20:34.779426", "stderr": "rm: cannot remove '/var/www/html/index.html': No such file or directory", "stderr_lines": ["rm: cannot remove '/var/www/html/index.html': No such file or directory"], "stdout": "", "stdout_lines": []}

TASK [create a file] ***********************************************************************************************************************************************************************
changed: [hap01.lab.rumah.lan]
changed: [hap02.lab.rumah.lan]

TASK [printing rescue status] **************************************************************************************************************************************************************
ok: [hap01.lab.rumah.lan] => {
    "msg": "rescue task was operated"
}
ok: [hap02.lab.rumah.lan] => {
    "msg": "rescue task was operated"
}

TASK [always write a message to logs] ******************************************************************************************************************************************************
changed: [hap01.lab.rumah.lan]
changed: [hap02.lab.rumah.lan]

TASK [always printing this message] ********************************************************************************************************************************************************
ok: [hap01.lab.rumah.lan] => {
    "msg": "this message is always printed"
}
ok: [hap02.lab.rumah.lan] => {
    "msg": "this message is always printed"
}

PLAY RECAP *********************************************************************************************************************************************************************************
hap01.lab.rumah.lan        : ok=5    changed=2    unreachable=0    failed=0    skipped=0    rescued=1    ignored=0   
hap02.lab.rumah.lan        : ok=5    changed=2    unreachable=0    failed=0    skipped=0    rescued=1    ignored=0   

kamal@TS-Kamal:~/github/ansible$ 
```