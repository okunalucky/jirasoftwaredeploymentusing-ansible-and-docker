# JIRA DEPLOYMENT USING ANISBLE AND DOCKER

<img width="841" height="581" alt="Image" src="https://github.com/user-attachments/assets/dea24528-96a2-42aa-8132-3c828090bfd7" />

- Creating the instances
- Create two instances, master node and worker node, in the master node, create a security group called master-sg and allow inbound traffic from IP. in the worker node, allow traffic from the master sg and from ip
- log into the two instances and rename the instances to easily identify them 

```sh
sudo su
sudo nano/etc/hostname
reboot
```
- Create ansible as a common user for the two servers and also set a password 

```sh
useradd ansible
passwd ansible 
```
- Allow password authentication and permit root log in
```sh
sudo nano /tc/ssh/seshd_config
restart ssh service 
sudo systemctl restart sshd
```

- Add ansible to the sudoers group
```sh
sudo nano/etc/sudoers
```
- Create a key pair in the master node such that ansible user can navigate to the worker node without password 
```sh
ssh-keygen -t rsa 
```
- Set permission 
```sh
chmod 700 ~/.ssh
copy the key pair generated 
ssh-copy-id -i ~/.ssh/my-key-pair.pub ec2-user@remote-server-ip
``` 
- Install ansible in the master node
```sh
sudo dnf install -y ansible-core
```
- Log in as a root user 
```sh
sudo su
```
- Navigate to the home directory of the root user
```sh
cd ~
cd /etc/ansible
sudo nano hosts
```
- Paste below content and edit Ip as per yours
```sh
  [webservers]
172.31.24.254
[webservers:vars]
ansible_user=ansible
ansible_ssh_private_key_file=/home/ansible/.ssh/id_rsa
```

- Ping the ip address to see if it will respond
```sh
sudo su ansible
cd ~
ansible all -m ping 
```
- Install docker in the worker node
```sh
sudo dnf install -y docker
sudo systemctl start docker
sudo systemctl enable docker
```

- Navigate to the home directory of the master node
```sh
cd ~
mkdir jira-docker
cd jira-docker
sudo nano docker-compose.yml
```

- Paste in the below content

```yaml
version: '3'
services:
  jira:
    image: atlassian/jira-software:latest  # Pull the latest Jira Software image
    ports:
      - "8080:8080"
    volumes:
      - jira-data:/var/atlassian/jira
    environment:
      - ATL_JDBC_URL=jdbc:postgresql://db/jiradb
      - ATL_JDBC_USER=jirauser
      - ATL_JDBC_PASSWORD=jirapassword

  db:
    image: postgres:9.6
    environment:
      - POSTGRES_DB=jiradb
      - POSTGRES_USER=jirauser
      - POSTGRES_PASSWORD=jirapassword
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  jira-data:
  db-data:
```  

- In the home directory of the ansible user
```sh
ansible-galaxy collection install community.docker --force
```
- Create the playbook
```sh
cd ~
mkdir ansible-playbook
cd ansible-playbook 
sudo nano deploy_jira.yml
```
- Paste below the content of the playbook

```yaml
---
- name: Deploy Jira via Docker Compose on Amazon Linux 2023
  hosts: webservers
  become: true
  vars:
    jira_project_dir: /opt/jira
    compose_version: "v2.27.1"
    compose_url: "https://github.com/docker/compose/releases/download/{{ compose_version }}/docker-compose-linux-x86_64"

  tasks:
    # 1. Install Docker engine
    - name: Install Docker engine
      ansible.builtin.dnf:
        name: docker
        state: present

    # 2. Ensure Docker service is started and enabled
    - name: Ensure Docker service is running
      ansible.builtin.service:
        name: docker
        state: started
        enabled: yes

    # 3. Create CLI plugin directory for Docker Compose v2
    - name: Create CLI plugin directory for Docker Compose v2
      ansible.builtin.file:
        path: /usr/local/lib/docker/cli-plugins
        state: directory
        mode: '0755'

    # 4. Install Docker Compose v2 plugin
    - name: Install Docker Compose v2 plugin
      ansible.builtin.get_url:
        url: "{{ compose_url }}"
        dest: /usr/local/lib/docker/cli-plugins/docker-compose
        mode: '0755'

    # 5. Create Jira project directory
    - name: Create Jira project directory
      ansible.builtin.file:
        path: "{{ jira_project_dir }}"
        state: directory
        mode: '0755'

    # 6. Copy your Compose file from control node to target node
    - name: Copy Docker Compose file from local to remote
      ansible.builtin.copy:
        src: "{{ lookup('env', 'HOME') }}/jira-docker/docker-compose.yml"
        dest: "{{ jira_project_dir }}/docker-compose.yml"
        mode: '0644'

    # 7. Deploy Jira stack using Docker Compose v2
    - name: Deploy Jira stack
      community.docker.docker_compose_v2:
        project_src: "{{ jira_project_dir }}"
        state: present
```
      

- In the same file path of the ansible playbook, you can now run the playbook
```yaml
ansible-playbook deploy_jira.yml
```
# EXECUTED PLAYBOOK
![Image](https://github.com/user-attachments/assets/839d7ab3-7388-4abb-8a4c-ab83a5bf9196)
<p>log in to aws console, open port 8080 on the worker node and run <code>@publicipaddress:8080</code> to complete the jira installation</p>

# JIRA SET UP

![Image](https://github.com/user-attachments/assets/2e765de9-9fc2-4d87-96cb-fc70ac57aba5)

