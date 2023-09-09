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

```yaml
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

