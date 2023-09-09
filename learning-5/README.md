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
