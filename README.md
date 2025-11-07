<h1>Creating the instances</h1>
<li>create two instances, master node and worker node, in the master node, create a security group called master-sg and allow inbound traffic from IP. in the worker node, allow traffic from the master sg and from ip
</li>
<h1>log into the two instances and rename the instances to easily identify them </h1>
<li><code>sudo su
sudo nano/etc/hostname
reboot</code></li>
<h1>create ansible as a common user for the two servers and also set a password </h1>
<li><code>useradd ansible
passwd ansible 
</code></li>
<h1>allow password authentication and permit root log in</h1>
<l1><code>sudo nano /tc/ssh/seshd_config
restart ssh service 
sudo systemctl restart sshd
</code></l1>
<h1>add ansible to the sudoers group </h1>
<li><code>sudo nano/etc/sudoers
</code></li>
<h1>create a key pair in the master node such that ansible user can navigate to the worker node without password </h1>
<l1><code>ssh-keygen -t rsa 
set permission 
chmod 700 ~/.ssh
copy the key pair generated 
ssh-copy-id -i ~/.ssh/my-key-pair.pub ec2-user@remote-server-ip
</code></l1> 
<h1>install ansible in the master node</h1>
<li><code>sudo dnf install -y ansible-core
</code></li>
<p>log in as a root user</p> 
<code>sudo su</code>code>
navigate to the home directory of the root user 
<code>
<code>cd ~
cd /etc/ansible
sudo nano hosts</code>code>
</code>
<code>
  [webservers]
172.31.24.254
[webservers:vars]
ansible_user=ansible
ansible_ssh_private_key_file=/home/ansible/.ssh/id_rsa</code>
<h1>ping the ip address to see if it will respond</h1>
<code>sudo su ansible
cd ~
ansible all -m ping 
</code>
<h1>install docker in the worker node</h1>
<code>sudo dnf install -y docker
sudo systemctl start docker
sudo systemctl enable docker
</code>
<h1>navigate to the home directory of the master node</h1>
<code>cd ~
mkdir jira-docker
cd jira-docker
sudo nano docker-compose.yml
</code>
<h1>this is the docker compose file</h1>
<code>
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
</code>
<p>in the home directory of the ansible user <code>ansible-galaxy collection install community.docker --force
</code></p>
<h3>create the playbook</h3>
<code>cd ~
mkdir ansible-playbook
cd ansible-playbook 
sudo nano deploy_jira.yml
</code>
<code>
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
</code>
