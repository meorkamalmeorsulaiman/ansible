# Ansible templating

We get a bit more dynamic with jinja template. Previously we learning about template but contain a static variable.

## List

We going to add an backend server pool `learning-7/vars/backendServers.yml` using list

```yaml
- backend_pool_group_web:
    - server: 192.168.100.210
    - server: 192.168.100.211
    - server: 192.168.100.212

- backend_pool_group_app:
    - server: 192.168.100.213
    - server: 192.168.100.214
    - server: 192.168.100.215
```
## Ansible variable

Create an ansible variable `learning-7/vars/app.yml`

```yaml
webapp:
  - frontend_name: myFirstFrontEnd
    backend_name: myFirstBackEnd
    frontend_port: 8080
    backend_port: 3000

  - frontend_name: mySecondFrontEnd
    backend_name: mySecondBackEnd
    frontend_port: 8081
    backend_port: 3001
```

## Jinja template

Create a dynamic template j2 `learning-7/templates/haproxy.cfg.j2`

```jinja
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


{% for web in webapp %}
#######################
#Frontend {{ web.frontend_name|e }}
#######################
frontend {{ web.frontend_name|e }}
    bind 0.0.0.0:{{ web.frontend_port|e }}
    default_backend {{ web.backend_name|e }}

#######################
#Backend {{ web.backend_name|e }}
#######################
backend {{ web.backend_name|e }}
{% for backendpool in backend_pool_group_web %}
    server {{ backendpool.server }} {{ backendpool.server|e }}:{{ web.backend_port|e }}
{% endfor %}

{% endfor %}
```

## Test 

Run the playbook using `ansible-playbook learning-7/deployHapJ2.yml --ask-pass`

```bash
sysadmin@ACCIO2:ansible$ ansible-playbook learning-7/deployHapJ2.yml --ask-pass
SSH password: 
BECOME password[defaults to SSH password]: 

PLAY [Deploy HAProxy] ***********************************************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************************************************************************************************************
ok: [192.168.100.221]
ok: [192.168.100.222]

TASK [Deploy HAP Configuration file] ********************************************************************************************************************************************************************************************************************************************
changed: [192.168.100.221]
changed: [192.168.100.222]

PLAY RECAP **********************************************************************************************************************************************************************************************************************************************************************
192.168.100.221            : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
192.168.100.222            : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

sysadmin@ACCIO2:ansible$ 
```

## Result

The config files is now a bit more dynamic

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


#######################
#Frontend myFirstFrontEnd
#######################
frontend myFirstFrontEnd
    bind 0.0.0.0:8080
    default_backend myFirstBackEnd

#######################
#Backend myFirstBackEnd
#######################
backend myFirstBackEnd
    server 192.168.100.210 192.168.100.210:3000
    server 192.168.100.211 192.168.100.211:3000
    server 192.168.100.212 192.168.100.212:3000

#######################
#Frontend mySecondFrontEnd
#######################
frontend mySecondFrontEnd
    bind 0.0.0.0:8081
    default_backend mySecondBackEnd

#######################
#Backend mySecondBackEnd
#######################
backend mySecondBackEnd
    server 192.168.100.210 192.168.100.210:3001
    server 192.168.100.211 192.168.100.211:3001
    server 192.168.100.212 192.168.100.212:3001

[root@hap01 ~]# 
```