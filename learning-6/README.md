# Ansible Templating

## Create Ansible variables

These variables will be use from the ansible playbook `learning-6/vars/app1.yml`

```yaml
---
frontend: myfirstFrontEnd
backend: myfirstBackEnd
backend_server_1: 192.168.100.210
backend_server_2: 192.168.100.211
backend_server_3: 192.168.100.212
backend_server_port: 3000
```

## Create the jinja template for HAProxy configuration file

Below will be the base config for HAProxy, this example will hold static variable

```bash
sysadmin@ACCIO2:ansible$ cat learning-6/templates/haproxy.cfg.j2 
###############
Default
###############
defaults
        mode tcp
        timeout client 10s
        timeout connect 5s
        timeout server 10s
        timeout http-request 10s
        log 127.0.0.1:514 local0
        option tcplog


##############
Frontend
##############

frontend {{ frontend }}
    bind 0.0.0.0:8080
    default_backend {{ backend }}

##############
Backend
##############
backend {{ backend }}
    server {{ backend_server_1 }} {{ backend_server_1 }}:3000 
    server {{ backend_server_3 }} {{ backend_server_3 }}:3000 
    server {{ backend_server_3 }} {{ backend_server_3 }}:3000
```

## Create playbook to deploy the configuration

The playbook will also refer back to j2 template and the variables that we are going to use

```bash
sysadmin@ACCIO2:ansible$ cat learning-6/deployHapJ2.yml 
---
- name: HAProxy Upgrades - Pre-Check
  hosts: hapnodns
  vars_files:
    - "vars/app1.yml"
  tasks:
  - name: Deploy HAP Configuration file
    template:
      src: ../templates/haproxy.cfg.j2
      dest: /tmp/haproxy.cfg
```

## Deploy the playbook

To run the playbook use `ansible-playbook learning-6/deployHapJ2.yml --ask-pass`

```bash
sysadmin@ACCIO2:ansible$ ansible-playbook learning-6/deployHapJ2.yml --ask-pass
SSH password: 
BECOME password[defaults to SSH password]: 

PLAY [Deploy HAProxy] ***********************************************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************************************************************************************************************
ok: [192.168.100.222]
ok: [192.168.100.221]

TASK [Deploy HAP Configuration file] ********************************************************************************************************************************************************************************************************************************************
changed: [192.168.100.222]
changed: [192.168.100.221]

PLAY RECAP **********************************************************************************************************************************************************************************************************************************************************************
192.168.100.221            : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
192.168.100.222            : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

sysadmin@ACCIO2:ansible$ 
```

## Validate the confguration

```bash
[root@hap01 ~]# cat /tmp/haproxy.cfg 
###############
Default
###############
defaults
        mode tcp
        timeout client 10s
        timeout connect 5s
        timeout server 10s
        timeout http-request 10s
        log 127.0.0.1:514 local0
        option tcplog


##############
Frontend
##############

frontend myfirstFrontEnd
    bind 0.0.0.0:8080
    default_backend myfirstBackEnd

##############
Backend
##############
backend myfirstBackEnd
    server 192.168.100.210 192.168.100.210:3000 
    server 192.168.100.212 192.168.100.212:3000 
    server 192.168.100.212 192.168.100.212:3000
[root@hap01 ~]# 
```