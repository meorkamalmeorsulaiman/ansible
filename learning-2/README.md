# Working with YAML

- Googled - it's a human-readable data serialization language
- Often used of writing configuration files

## Identation

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

- Validate the syntax with `ansible-playbook --syntax-check learning-2/deployhttpd.yml`

```shell
kamal@TS-Kamal:~/github/ansible$ ansible-playbook --syntax-check learning-2/deployhttpd.yml 

playbook: learning-2/deployhttpd.yml
```