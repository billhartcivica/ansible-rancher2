# rancher2-prep

## Ansible files for prepping a group of machines to function in a Rancher2 environment

To import a group of machines into a Rancher2 environment, target hosts must first be prepared. The host specification should be as follow:

- Base O/S: Centos 7
- Setup root user with password
- Create an entry for each host in the /etc/hosts file of the Ansible host
- Run the 'ssh-copy-id root@<host>' command to copy your SSH key from the Ansible host to the target host(s)

Once the base installation is complete, the prep.yml playbook will carry out the following steps:
- User: Create the ansible user (subsequent Ansible playbook runs can use this account!)
- The sudoers file is amended so that members of the 'wheel' group can sudo without login
- The 'ansible' user is made a member of the 'wheel' and 'docker' group.
- Selinux is disabled and firewalld is stopped/disabled.
- Docker is installed, enabled and started.
- The ansible contol host should copy it's ssh key to the 'ansible' user on the target hosts (eg: ssh-copy-id ansible@remotehost)
- The ansible host requires an update to it's /etc/hosts file to resolve all the target hosts in the cluster before applying the Ansible script.
- The 'inventory' file in this repo needs to be updated with the hostnames of the target cluster hosts.
- Each target host should have the /etc/hosts file updated with the IP/machinename of themselves and their neighbouring hosts.

Once the above steps are completed, the final part of the preparation can be applied using the 'run.sh' script. This runs ansible-playbook agains the inventory file, connecting to each cluster member as 'ansible' and applying the 'prep.yml' playbook.

Note: one of the machines prepared above should serve as the Rancher2 host. This is run via a simple Docker command:
```
'docker run -d --restart=unless-stopped -p 80:80 -p 443:443 -v /host/certs:/container/certs -e SSL_CERT_DIR="/container/certs" rancher/rancher:latest'
```
Alternately, run without the cert mapping, assuming you don't want to include any pre-defined certificates:
```
'docker run -d --restart=unless-stopped -p 80:80 -p 443:443 rancher/rancher'
```
Update: The above commands are now replaced by a docker-compose.yml file located in the 'files' folder of this repository. This is copied to the Rancher control host and run from there.

## Post-prep tasks:
Once the Ansible playbook has been run against the target hosts, log into the ansible controller host by pointing to the hostname/IP on port 8080 (this will redirect to port 8443) and enter the username/password (admin/admin). The service will prompt you to change the admin password. Change it and once done you have logged into the system.

Click on the 'Global' menu and click once on the 'Add Cluster' button to the right.

Choose 'Custom' from the list of cluster types and enter a cluster name. Once done, scroll down and click on the 'Next' button.

The next window will give you the node options and the command you need to run on each node for it to assume that role. Choose 'etcd' and 'Control Plane' or 'Worker' to amend the command accordingly. Copy the command from the pane below and paste it into a shell session on each node. Wait for the cluster to complete building and once all nodes are shown in an 'Active' state, the cluster is ready for use.
