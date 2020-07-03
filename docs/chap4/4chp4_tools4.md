# 第四节 Jenkins集成Ansibe实战手册 ——— BB Mobile（2018）

### Description: 

Ansible setup functions and aws ec2 manipulation functions for Jenkins master and agents

## 1、Getting Started

* `install pip and pyenv` **[version:2.7.9]**
* `pip install ansible in pyenv`
* absoulte path:  ```/home/ubuntu/pyenv/versions/ansible/bin```
* check ansible version: 

```
/home/ubuntu/pyenv/versions/ansible/bin/ansible --version
ansible 2.5.1
  config file = None
  configured module search path = [u'/home/ubuntu/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /home/ubuntu/pyenv/versions/2.7.9/envs/ansible/lib/python2.7/site-packages/ansible
  executable location = /home/ubuntu/pyenv/versions/ansible/bin/ansible
  python version = 2.7.9 (default, Apr 28 2018, 06:18:59) [GCC 4.8.4]
```

* enter virtual env of ansible, you dont have use absolute path

```
source ~/pyenv/versions/ansible/bin/activate
```

* use ansible directly

## 2、 Ansible files layout

ansible `agent_playbook` laying out roles, inventories ans playbooks

```
ansible-foobar/
├── cloud
│   ├── ec2.ini
│   ├── ec2.py
│   └── group_vars
├── roles
|   ├── ansible_playbook_name
|   |   |__ files --- files
|   |   |__ tasks --- main.yml
|   |   |__ templates --- tempalte
│   ├── base
|   |   |__ files --- jenkins_master_pub
|   |   |__ tasks --- main.yml
|   |   |__ templates --- 01-warning.j2
│   └── agents_name_for_setup
|   └── function_name_for_ec2
├── handlers
│   └── main.yml
├── localhost
├── site.yml
├── agent_name.yml
└── ec2_function.yaml
```

### `cloud/ec2.ini`

```
[ec2]
# To exclude RDS instances from the inventory, uncomment and set to False.
rds = False
# To exclude ElastiCache instances from the inventory, uncomment and set to False.
elasticache = False
regions = us-east-1
destination_variable = private_dns_name
vpc_destination_variable = private_ip_address
```


## 3、Run the ansible playbook for setup

how to seup jenkins master and agent with ansible playbook

```ansible-playbook -i cloud/ec2.py android_sdk_builder.yml -vvv```

```ansible-playbook -i cloud/ec2.py android_sdk_builder.yml  ```

```ansible-playbook -i cloud/ec2.py agents_name.yml ```

## 4、Run the ansible playbook for ec2 function

how to manipulate ec2 instance or instances with ansible playbook

```
/home/ubuntu/pyenv/versions/ansible/bin/ansible-playbook agents_playbook/ec2_ansible_function.yaml -i agents_playbook/localhost -t stop-one -e ONE_INSTNACE_NAME=instance_name -vvv
```

for example:

```
ansible-playbook agents_playbook/ec2_ansible_function.yaml -i agents_playbook/localhost -t tags_name  -e  additional variables as key=value -vvv
```

### Reference

* ansible playbook command line
```https://docs.ansible.com/ansible/2.4/ansible-playbook.html```

* ansible agent playbook layout
```https://leucos.github.io/ansible-files-layout```

* README-Template.md
```https://gist.github.com/PurpleBooth/109311bb0361f32d87a2```