# jirasoftwaredeploymentusing-ansible-and-docker
this project describe the step by step approach of deploying jira using ansible playbook
<li>Launch an amazon linux 2, t2.micro instance and call it Ansible-control, create a security group ansible-control-sg, set an inbound rule to allow ssh traffic from IP</li>
<li>Launch two instances (amazon linux 2, t2.micro) and call it worker-node1 and worker-node 2, set an inbound rule to allow ssh trafocfrom the ansible-control-sg and from an IP</li>
<li>Log into the three instances using git bash and rename them properly for easy identification</li>
<code>sudo su 
nano /etc/hostname
</code>
<li>For the three servers, create a common user called ansible, set a common password for the user, allow for password authentication and permit root log in</li>
<code>sudo su 
useradd ansible   
passwd ansible
</code>
<li>change password authentication to yes and uncomment permit root log in</li>
<code>sudo nano /etc/ssh/sshd_config</code>
<li>Add ansible to the sudoers group in each server</li>
<code>sudo nano /etc/sudoers</code>
<li>generate a key pair in the ansible-control and copy it to the worker nodes such that ansible user can ssh into any of the nodes without requesting for password</li>
<code>cd ~
ssh-keygen –t rsa
sudo chmod 700 /home/ansible/.ssh
ssh-copy-id ansible@privateipoftheworkernode
</code>
<li>Install Ansible package in the ansible-control </li>
<code>sudo dnf install ansible –y</code>
<li>Install Docker in the worker –nodes</li>
<code>sudo yum install docker –y
sudo service docker start
sudo systemctl enable docker
sudo systemctl status docker
</code>
<li>Update the inventory file in the Ansible-control so that Ansible can identify the nodes it communicates with when the playbook runs</li>
<code>cd  /etc/ansible/hosts</code>  -update the hosts here with the ip address of the worker nodes
<li>then create an ansible.cfg file and direct the inventory to the path of /etc/ansible/hosts</li>
<li>Create directories and files in the control node for the docker compose file</li>
<code>mkdir  ~/jira-docker
cd  ~/jira-docker
sudo nano docker-compose.yml
</code>
<li>Create directories and files in the control node for the ansible playbook</li>
<code>mkdir ~/ansible-playbooks
cd ~/ansible-playbooks
sudo nano deploy-jira.yml
</code> 
<li>Ensure you’re in the right file path cd/ansible-playbooks/deploy-jira.yml, run the playbook</li>
<code>ansible-playbook deploy-jira.yml</code>
<li>You can view and set up the jira app using the public ip address of any of the worker node on port 8080 and complete the setup</li>

