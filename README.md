## Production AWS Setup for Tower
Credentials in order to communicate with AWS instances should be set up in the following manner:
1. Set up your project.
    * Name your project.
    * Set the SCM Type to Git.
    * Set the repo to this repo that you cloned.
2. Set up the AWS Inventory.
    * Name it accordingly under the *Details* tab.
    * From the *Sources* tab:
      * Click Add Source.  
        * Set Name to reference later on.
        * Source should be `Amazon EC2`.
        * Set the *Credential* to the teacher supplied `AWS RO Credential`.
        * Region should be *US East (Northern Virginia)*.
        * Set the *Instance Filters* to `tag:owner=twardzin@redhat.com`.  Or, whatever your resource identifer is.
    * TODO: Put section here about adding the groups pointing to AWS Groups for reference later on in the Provision_AWS.yml playbook.
3. Set up credentials in order for Ansible Tower to communicate to 3 Tier Resources.
    * Acquire the GUID setup from OpenTLC.
    * `ssh` to your bastion machine:
      ```
      [laptop ]$ ssh -i ~/.ssh/id_rsa twardzin-redhat.com@bastion.9ce4.example.opentlc.com
      ```
    * Get the key information from the bastion:
      ```
      sudo cat /root/.ssh/${GUID}key.pem 
      ```
    * In Ansible Tower, go to Settings (Gear) -> Credentials -> Create a New Credential
    * Fill the following out:
      * Name
      * Description
      * Organization should be *Default*.
      * Credential Type is *Machine*.
      * Selecting *Machine* brings up additional information to fill out:
        * Username should be `ec2-user`.
        * Password should be blank.
        * SSH Private Key should now get the key from `sudo cat /root/.ssh/${GUID}key.pem`.
        * Privilege Escalation Method is set to *sudo*.
      * When all the information is filled out, click *Save*.
4. Everything is now ready for setting up a *Template* in order to deploy the Ansible Playbook for provisioning AWS.
    * Click the *Templates* tab.  Click *Add* and then click `Job Template`.
    * Set the *Name* to reference later on.  Description as well.
    * *Job Type* should be set to `Run`.
    * *Inventory* should be set to what you named your filtered inventory to.
    * *Project* should be set to your project name created in step #1.
    * *Playbook* should be set to the correct playbook from the git repo cloned in step #1.  In this case, set it to `Provision_AWS.yml`.
    * *Credential* should be set to what you named your credentials in step #3.
    







# >>Previous Readme from other build...probably should integrate into our readme.
main.yaml

> Configure SSH keys on Jumpbox

```
ssh -i ./.ssh/yourprivatekey userid@workstation-${GUID}.rhpds.opentlc.com
wget http://www.opentlc.com/download/ansible_bootcamp/openstack_keys/openstack.pub
cat openstack.pub  >> ~/.ssh/authorized_keys
```

> Configure laptop or bastion host for ssh proxy

```
wget http://www.opentlc.com/download/ansible_bootcamp/openstack_keys/openstack.pem -O ~/.ssh/openstack.pem
chmod 400 ~/.ssh/openstack.pem


cat << EOF > /etc/ansible/openstack_ssh_config
Host workstation-${GUID}.rhpds.opentlc.com
 Hostname workstation-${GUID}.rhpds.opentlc.com
 IdentityFile ~/.ssh/openstack.pem
 ForwardAgent yes
 User cloud-user
 StrictHostKeyChecking no
 PasswordAuthentication no

Host 10.10.10.*
 User cloud-user
 IdentityFile ~/.ssh/openstack.pem
 ProxyCommand ssh -F ~/.ssh/openstack_ssh_config cloud-user@workstation-${GUID}.rhpds.opentlc.com -W %h:%p -vvv
 StrictHostKeyChecking no
EOF


cat << EOF > osp_test_inventory
[jumpbox]
workstation-${GUID}.rhpds.opentlc.com ansible_ssh_user=root ansible_ssh_private_key_file=~/.ssh/openstack.pem
EOF

ansible -i osp_test_inventory all -m ping


ansible -i osp_test_inventory jumpbox -m os_user_facts -a cloud=ospcloud -v

cat << EOF > ssh_ansible.yaml
- hosts: localhost
  tasks:
  - name: Add ssh_args in ansible.cfg to point to the user's SSH config
    lineinfile:
      path: /etc/ansible/ansible.cfg
      insertafter: '^#ssh_args '
      line: 'ssh_args = -F /etc/ansible/openstack_ssh_config -o ControlMaster=auto -o ControlPersist=5m -o LogLevel=QUIET'
EOF
```


 
Step 1: Deploy Insfrastructure on OSP 10

>roles/osp-instances/vars/frontend.yaml
```
instance_name: frontend
group: frontends
deployment: dev
security_group_name: frontend_servers
```

>roles/osp-instances/vars/app1.yaml
```
instance_name: app1
group: apps
deployment: dev
security_group_name: app_servers
```

>roles/osp-instances/vars/app2.yaml
```
instance_name: app2
group: apps
deployment: dev
security_group_name: app_servers
```

>roles/osp-instances/vars/db.yaml
```
instance_name: db
group: appdbs
deployment: dev
security_group_name: db_servers
```
Step 2: Configure Instances 

Step 3: Deploy example APP
