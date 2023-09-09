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
- Let say if the 1st tasks failed then you don't want to run 2nd task but proceed with 3rd task
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