# Multiplay Playbook

- A playbook that consists multiple play
- This used to setup the entire managed environment
- Similar to `compose` and `stack`

## Mutiplay example 

- The playbook will setup httpd service
- The 2nd play will use to test the web service accessibility
- Example:

```yaml
---
---
- name: install start and enable httpd
  hosts: hap
  tasks:
  - name: Install web package
    yum:
      name: httpd
      state: latest

  - name: Start and enable web service
    service:
      name: httpd
      state: started
      enabled: yes

- name: Test web accessibility
  hosts: localhost
  tasks:
  - name: Test Web Service
    uri:
     url:
      - http://hap01.lab.rumah.lan
      - http://hap02.lab.rumah.lan
```

- Validate that the syntax with `ansible-playbook learning-3/deployWebService.yml --syntax-check`

```bash
kamal@TS-Kamal:~/github/ansible$ ansible-playbook learning-3/deployWebService.yml --syntax-check 

playbook: learning-3/deployWebService.yml
kamal@TS-Kamal:~/github/ansible$ 
```

- Perform dry run `ansible-playbook -C learning-3/deployWebService.yml --ask-pass`

```bash
kamal@TS-Kamal:~/github/ansible$ ansible-playbook -C learning-3/deployWebService.yml --ask-pass
SSH password: 
BECOME password[defaults to SSH password]: 

PLAY [install start and enable httpd] ****************************************************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************************************************************
ok: [hap02.lab.rumah.lan]
ok: [hap01.lab.rumah.lan]

TASK [Install web package] ***************************************************************************************************************************************************************************
changed: [hap01.lab.rumah.lan]
changed: [hap02.lab.rumah.lan]

TASK [Start and enable web service] ******************************************************************************************************************************************************************
fatal: [hap02.lab.rumah.lan]: FAILED! => {"changed": false, "msg": "Could not find the requested service httpd: host"}
fatal: [hap01.lab.rumah.lan]: FAILED! => {"changed": false, "msg": "Could not find the requested service httpd: host"}

PLAY RECAP *******************************************************************************************************************************************************************************************
hap01.lab.rumah.lan        : ok=2    changed=1    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0   
hap02.lab.rumah.lan        : ok=2    changed=1    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0  
```

- Notice that the failed task, this is expected as the service haven't been installed in the managed hosts
- Execute the playbook `ansible-playbook learning-3/deployhttpd.yml --ask-pass`

```bash
kamal@TS-Kamal:~/github/ansible$ ansible-playbook learning-3/deployWebService.yml --ask-pass
SSH password: 
BECOME password[defaults to SSH password]: 

PLAY [install start and enable httpd] ****************************************************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************************************************************
ok: [hap01.lab.rumah.lan]
ok: [hap02.lab.rumah.lan]

TASK [Install web package] ***************************************************************************************************************************************************************************
changed: [hap01.lab.rumah.lan]
changed: [hap02.lab.rumah.lan]

TASK [Start and enable web service] ******************************************************************************************************************************************************************
changed: [hap01.lab.rumah.lan]
changed: [hap02.lab.rumah.lan]

PLAY [Test web accessibility] ************************************************************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************************************************************
ok: [localhost]

TASK [Test Web Service] ******************************************************************************************************************************************************************************
fatal: [localhost]: FAILED! => {"changed": false, "elapsed": 0, "msg": "Status code was -1 and not [200]: Request failed: <urlopen error unknown url type: ['http>", "redirected": false, "status": -1, "url": "['http://hap01.lab.rumah.lan', 'http://hap02.lab.rumah.lan']"}

PLAY RECAP *******************************************************************************************************************************************************************************************
hap01.lab.rumah.lan        : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
hap02.lab.rumah.lan        : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
localhost                  : ok=1    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0  
```
