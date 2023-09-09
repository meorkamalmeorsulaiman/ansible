# ansible
This is the place for my ansible notes

## Set environment variable

`export ANSIBLE_CONFIG=./ansible.ini`

## Initial Test

`ansible -m ping -i inventory.yml all --ask-pass -v`

```
kamal@TS-Kamal:~/github/ansible$ ansible -i inventory.yml all -m ping --ask-pass -v
Using /home/kamal/github/ansible/ansible.ini as config file
SSH password: 
BECOME password[defaults to SSH password]: 
hap01.lab.rumah.lan | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
192.168.100.222 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
hap02.lab.rumah.lan | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
192.168.100.221 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
```


## Test the playbook

