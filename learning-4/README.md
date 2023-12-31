# Task Control with Ansible

## Loops and Items

- We can use list for some of the ansible module
- Many module don't
- So we need to use loop to iterate variable or item over
- Example:

```yaml
---
- name: install and start services
  hosts: hap
  tasks:
   - name: install packages
     yum:
      name:
       - httpd
       - keepalived
      state: latest

   - name: start the services
     service:
      name: "{{ item }}"
      state: started
      enabled: yes
     loop:
      - httpd
      - keepalived
```

- Execute the playbook `ansible-playbook learning-4/playbooks/simpleLoop.yml --ask-pass`

```yaml
kamal@TS-Kamal:~/github/ansible$ ansible-playbook learning-4/playbooks/simpleLoop.yml --ask-pass
SSH password: 
BECOME password[defaults to SSH password]: 

PLAY [install and start services] ********************************************************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************************************************************
ok: [hap02.lab.rumah.lan]
ok: [hap01.lab.rumah.lan]

TASK [install packages] ******************************************************************************************************************************************************************************
changed: [hap01.lab.rumah.lan]
changed: [hap02.lab.rumah.lan]

TASK [start the services] ****************************************************************************************************************************************************************************
changed: [hap01.lab.rumah.lan] => (item=httpd)
changed: [hap02.lab.rumah.lan] => (item=httpd)
changed: [hap01.lab.rumah.lan] => (item=keepalived)
changed: [hap02.lab.rumah.lan] => (item=keepalived)

PLAY RECAP *******************************************************************************************************************************************************************************************
hap01.lab.rumah.lan        : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
hap02.lab.rumah.lan        : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

kamal@TS-Kamal:~/github/ansible$ 
```

- Above, the `yum` module supports YAML list
- While the `service` module we use loop
- When using list, you need to check whether the module support list
- If it doesn't supported then use loop with `{{ item }}` as the value/variable to the key
- The `{{ item }}` is system internal variable

## Loop with variable

- You can provide a variable for the loop reference
- The variable defined/declare in the play header

```yaml
---
- name: install and start services
  hosts: hap
  vars:
   services:
    - keepalived
    - httpd
  tasks:
   - name: install packages
     yum:
      name:
       - keepalived
       - httpd 
      state: latest
  
   - name: start the services
     service:
      name: "{{ item }}"
      state: started
      enabled: yes
     loop: "{{ services }}"
```

- Execute the playbook using `ansible-playbook learning-4/playbooks/loopWithVariable.yml --ask-pass`
- The variable can also be include in `host/hostgroup`

```bash
kamal@TS-Kamal:~/github/ansible$ ansible-playbook learning-4/playbooks/loopWithVariable.yml --ask-pass 
SSH password: 
BECOME password[defaults to SSH password]: 

PLAY [install and start services] ********************************************************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************************************************************
ok: [hap02.lab.rumah.lan]
ok: [hap01.lab.rumah.lan]

TASK [install packages] ******************************************************************************************************************************************************************************
changed: [hap02.lab.rumah.lan]
changed: [hap01.lab.rumah.lan]

TASK [start the services] ****************************************************************************************************************************************************************************
changed: [hap01.lab.rumah.lan] => (item=keepalived)
changed: [hap02.lab.rumah.lan] => (item=keepalived)
changed: [hap02.lab.rumah.lan] => (item=httpd)
changed: [hap01.lab.rumah.lan] => (item=httpd)

PLAY RECAP *******************************************************************************************************************************************************************************************
hap01.lab.rumah.lan        : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
hap02.lab.rumah.lan        : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

- Execute playbook to remove the services `ansible-playbook learning-4/playbooks/loopRemoveServices.yml --ask-pass`

## Loop with Multivalued Variables

- Item can be a simple list, it can also represent multiple variable.
- Example, a variable files lists or users:

```yaml
users:
  - username: user01
    homedir: /home/user01
    shell: /bin/bash
    groups: wheel
  - username: user02
    homedir: /home/user02
    shell: /bin/bash
    groups: users
  - username: user03
    homedir: /home/user03
    shell: /bin/bash
    groups: users
```

- The the playbook take the variable form the multivalue variable file.
- Execute the playbook using `ansible-playbook learning-4/playbooks/loopWithMultiVar.yml --ask-pass`

```bash
kamal@TS-Kamal:~/github/ansible$ ansible-playbook learning-4/playbooks/loopWithMultiVar.yml --ask-pass 
SSH password: 
BECOME password[defaults to SSH password]: 

PLAY [create users using a loop from a list] *********************************************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************************************************************
ok: [hap01.lab.rumah.lan]

TASK [create users] **********************************************************************************************************************************************************************************
changed: [hap01.lab.rumah.lan] => (item={'username': 'user01', 'homedir': '/home/user01', 'shell': '/bin/bash', 'groups': 'wheel'})
changed: [hap01.lab.rumah.lan] => (item={'username': 'user02', 'homedir': '/home/user02', 'shell': '/bin/bash', 'groups': 'users'})
changed: [hap01.lab.rumah.lan] => (item={'username': 'user03', 'homedir': '/home/user03', 'shell': '/bin/bash', 'groups': 'users'})

PLAY RECAP *******************************************************************************************************************************************************************************************
hap01.lab.rumah.lan        : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

kamal@TS-Kamal:~/github/ansible$ 
```

- Remove the users created using `ansible-playbook learning-4/playbooks/loopRemoveUsers.yml --ask-pass -C`
- Earlier version of Ansible, you can use `with_keyword` instead of `loop`
  - There are severals options:
    - `with_items` - Used like `loop`
    - `with_file` - Used to iterate over a list of filenames
    - `with_sequence` - Used to generate a list of values based on numeric sequence
