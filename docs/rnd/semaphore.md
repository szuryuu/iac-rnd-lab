**SEMAPHORE**

**Make a SSH Key “Empty”**  
because semaphore cannot entry passphrase

| cp \~/.ssh/azure\_vm\_key\_dev\_test1 \~/.ssh/azure\_vm\_key\_dev\_test1\_nopass chmod 600 \~/.ssh/azure\_vm\_key\_dev\_test1\_nopass ssh-keygen \-p \-f \~/.ssh/azure\_vm\_key\_dev\_test1\_nopass \-N "" |
| :---- |

**Running Semaphore in Local**

| docker run \-d \\                \--name semaphore \\                \--network host \\                \-e SEMAPHORE\_DB\_DIALECT=bolt \\                \-e SEMAPHORE\_ADMIN=admin \\                \-e SEMAPHORE\_ADMIN\_PASSWORD=changeme \\                \-e SEMAPHORE\_ADMIN\_NAME=Admin \\                \-e SEMAPHORE\_ADMIN\_EMAIL=admin@localhost \\                \-e ANSIBLE\_LOCAL\_TEMP=/tmp \\                \-e SEMAPHORE\_TMP\_PATH=/tmp \\                \-e ANSIBLE\_GALAXY\_CACHE\_DIR=/tmp \\                \-e ANSIBLE\_GALAXY\_SERVER\_CACHE\_PATH=/tmp \\                \-v \~/semaphore-data:/tmp/semaphore \\                \-v \~/Sz/projects/devops/PIaC/ansible:/ansible \\                \-v \~/semaphore-data/ssh/id\_rsa:/etc/semaphore/id\_rsa:ro \\                \-v \~/.ssh/config:/etc/semaphore/ssh\_config:ro \\                semaphoreui/semaphore:latest |
| :---- |
| ssh\_config Host ssh.dev.azure.com   User git   HostName ssh.dev.azure.com   IdentityFile \~/.ssh/id\_rsa\_azure   IdentitiesOnly yes Host boundary   HostName 4.193.226.1   User adminuser   IdentityFile /etc/semaphore/id\_rsa   StrictHostKeyChecking no   UserKnownHostsFile /dev/null   ServerAliveInterval 30   ServerAliveCountMax 5   ConnectTimeout 60 Host vm-dev   HostName 10.1.1.4   User adminuser   IdentityFile /etc/semaphore/id\_rsa   ProxyJump boundary   StrictHostKeyChecking no   UserKnownHostsFile /dev/null   ServerAliveInterval 30   ServerAliveCountMax 5   ConnectTimeout 60 |

**Inventory**

| \[dev\] vm-dev ansible\_host=vm-dev ansible\_user=adminuser ansible\_ssh\_common\_args='-F /etc/semaphore/ssh\_config' |
| :---- |


