# TP 3 - Commands_Ansible

## Useful commands
### Ansible with python 3.11
`python3.11 -m venv ansible`

### Activate the virtual environment
`source ansible/bin/activate`

### Check python version 
`python --version`

### Install ansible
`python3 -m pip install ansible`

### Check ansible version
`ansible --version`

### Ugrade ansible
`python3 -m pip install --upgrade ansible`  

### We give the read only permission to the key
`chmod 400 ./id_rsa`

### We connect to the server
`ssh -i ./id_rsa cento@julien.perbet.takima.cloud`
#### We need to validate the fingerprint

# TP 3 - Ansible
## setup.yml configuration
```yaml
all:
  vars:
    ansible_user: centos
    ansible_ssh_private_key_file: ~/.ssh/id_rsa_takima
  children:
    prod:
      hosts: julien.perbet.takima.cloud
```
### After this command `ansible all -i inventories/setup.yml -m ping` we have :
# IMAGE

### We use `ansible all -i inventories/setup.yml -m setup -a "filter=ansible_distribution*"` to get the distribution of the server:
# IMAGE

### Now we delete apache with `ansible all -i inventories/setup.yml -m yum -a "name=httpd state=absent" --become`:
# IMAGE

## playbook.yml first configuration
```yaml
- hosts: all
  gather_facts: false
  become: true

  tasks:
   - name: Test connection
     ping:
```

### We use `ansible-playbook -i inventories/setup.yml playbook.yml` to test the connection:
![](/tp3/img/connection_playbook_ansible.png)

## playbook.yml second configuration
```yaml
- hosts: all
  gather_facts: false
  become: true

# Install Docker
  tasks:

  - name: Install device-mapper-persistent-data
    yum:
      name: device-mapper-persistent-data
      state: latest

  - name: Install lvm2
    yum:
      name: lvm2
      state: latest

  - name: add repo docker
    command:
      cmd: sudo yum-config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo

  - name: Install Docker
    yum:
      name: docker-ce
      state: present

  - name: Install python3
    yum:
      name: python3
      state: present

  - name: Install docker with Python 3
    pip:
      name: docker
      executable: pip3
    vars:
      ansible_python_interpreter: /usr/bin/python3

  - name: Make sure Docker is running
    service: name=docker state=started
    tags: docker
```

### We use `ansible-playbook -i inventories/setup.yml playbook.yml` to install docker:
# IMAGE







