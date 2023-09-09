# Playbook

## Element

- Playbook - consists of plays, written in yaml
- Play that will target to specific hosts
- In each play, there are tasks specified

## Format

- It begin with `---` at the beginning at of the file
- Target host specify in the play
- After that you will specify the tasks
- There items in a playbook:
  - `---`
  - Target host
  - Name of the play
  - Task
- Each item in the list begin with
- Indentation:
  - Element at the same level must have the same indentation
  - Item that are child and another element are indented more that the parent
  - Indentation created using space

## Example of playbook

- the example playbook `learning-1/httpd.yml` installed httpd on one of the target host `hap01`
- Use `ansible-playbook` command to execute the playbook `ansible-playbook learning-1/httpd.yml --ask-pass`

```
kamal@TS-Kamal:~/github/ansible$ ansible-playbook learning-1/httpd.yml --ask-pass
SSH password: 
BECOME password[defaults to SSH password]: 

PLAY [install start and enable httpd] ****************************************************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************************************************************
ok: [hap01.lab.rumah.lan]

TASK [install package] *******************************************************************************************************************************************************************************
changed: [hap01.lab.rumah.lan]

TASK [start and enable service] **********************************************************************************************************************************************************************
changed: [hap01.lab.rumah.lan]

PLAY RECAP *******************************************************************************************************************************************************************************************
hap01.lab.rumah.lan        : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

kamal@TS-Kamal:~/github/ansible$ 
```

- By default firewalld is enable, we can create a playbook to turn off firewalld
- Use `ansible-playbook` command to execute the playbook `ansible-playbook learning-1/firewalld.yml --ask-pass`

```
kamal@TS-Kamal:~/github/ansible$ ansible-playbook learning-1/firewalld.yml --ask-pass
SSH password: 
BECOME password[defaults to SSH password]: 

PLAY [Stop and disable firewall in hap01 and hap02] **************************************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************************************************************
ok: [hap02.lab.rumah.lan]
ok: [hap01.lab.rumah.lan]

TASK [Stop and disable firewalld] ********************************************************************************************************************************************************************
changed: [hap01.lab.rumah.lan]
changed: [hap02.lab.rumah.lan]

PLAY RECAP *******************************************************************************************************************************************************************************************
hap01.lab.rumah.lan        : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
hap02.lab.rumah.lan        : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

kamal@TS-Kamal:~/github/ansible$ nc -vz hap01.lab.rumah.lan 80
Connection to hap01.lab.rumah.lan (192.168.100.221) 80 port [tcp/http] succeeded!
```