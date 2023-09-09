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

Example - Test multiple condition:

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