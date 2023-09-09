# Working with YAML

- Googled - it's a human-readable data serialization language
- Often used of writing configuration files

## Indentation

- Indent identify relations between parts of the config
- Rule of thumb you must use space and not tabs

## Key-Value Pairs

- Also known as dictionaries
- Example of key-value pair `name: httpd`
- It can be defined in two ways:
  - `key: value`
  - `key=value`
  - The first method is preferred
- If an object has be multiple key-vaule pairs, it can be define in a line
- Example:

```yaml
---
- name: deploy httpd
  hosts: hap01
  tasks:
   - name: install httpd
     yum: name=httpd
   
   - name: enable httpd
     service: name=httpd enabled=true
```

## Verify syntax

- Validate the syntax with `ansible-playbook --syntax-check learning-2/deployhttpd.yml`

```shell
kamal@TS-Kamal:~/github/ansible$ ansible-playbook --syntax-check learning-2/deployhttpd.yml 

playbook: learning-2/deployhttpd.yml
```

## YAML Lists

- Key can have one or multiple value
- When you use a module like `yum`, you can list packages name to a key
- Example:

```yaml
---
- name: using lists
  hosts: hap
  tasks:
  - name: install packages
    yum:
     name:
     - nmap
     - httpd
     - keepalived
     state: latest
```

- Validate the syntax is correct `ansible-playbook learning-2/installpackages.yml --syntax-check`

```bash
kamal@TS-Kamal:~/github/ansible$ ansible-playbook learning-2/installpackages.yml --syntax-check

playbook: learning-2/installpackages.yml
kamal@TS-Kamal:~/github/ansible$ 
```

## YAML Strings

- You have multiple way to describe/notation stings in YAMl file:

```txt
Line of string
"Line of string"
'Line of string'
```

- Two ways to use strings in yaml by using `|` to take over all newline
- You can use `>` for splitting text over multiple line

## Dry Run

- Using dry run, ansible will show the changes without applying the change
- Use `-C` option to perform dry run on `ansible-playbook`
- Execute the dry run `ansible-playbook -C learning-2/deployhttpd.yml --ask-pass`

```bash
kamal@TS-Kamal:~/github/ansible$ ansible-playbook -C learning-2/deployhttpd.yml --ask-pass
SSH password: 
BECOME password[defaults to SSH password]: 

PLAY [deploy httpd] **********************************************************************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************************************************************
ok: [hap01.lab.rumah.lan]

TASK [install httpd] *********************************************************************************************************************************************************************************
changed: [hap01.lab.rumah.lan]

TASK [enable httpd] **********************************************************************************************************************************************************************************
fatal: [hap01.lab.rumah.lan]: FAILED! => {"changed": false, "msg": "Could not find the requested service httpd: host"}

PLAY RECAP *******************************************************************************************************************************************************************************************
hap01.lab.rumah.lan        : ok=2    changed=1    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0  
```
