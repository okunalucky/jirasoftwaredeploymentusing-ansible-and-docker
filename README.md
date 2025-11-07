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
<link>ssh-keygen -t rsa </link>
