## Step #1 - QA OpenStack Setup for Tower
1. On your laptop, provision the workstation to accept OpenStack related configurations.
  * On workstation node issue the following commands: 
    * `wget http://www.opentlc.com/download/ansible_bootcamp/openstack_keys/openstack.pub`
    * `cat openstack.pub  >> /home/cloud-user/.ssh/authorized_keys`
    * `wget https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm`
    * `sudo yum -y install 'ls *epel*.rpm'`
    * `sudo yum install -y python python-devel python-pip gcc ansible`
    * `sudo pip install shade`
    * `sudo mkdir /etc/openstack`
    * Create configuration file:
      ```
      sudo cat << EOF > /etc/openstack/clouds.yaml
        clouds:
          ospcloud:
            auth:
              auth_url: http://192.168.0.20:5000/
              password: r3dh4t1!
              project_name: admin
              username: admin
            identity_api_version: '3.0'
            region_name: RegionOne
        ansible:
          use_hostnames: True
          expand_hostvars: False
          fail_on_errors: True
        EOF
        ```
    * Create additional configuration file:
      ```
      sudo cat << EOF > /etc/openstack/osp_image.yml
        - hosts: localhost
          become: yes
          gather_facts: false
          tasks:
          - name: download RHEL image
            get_url:
              url: http://www.opentlc.com/download/osp_advanced_networking/rhel-guest-image-7.2-20151102.0.x86_64.qcow2
            dest: /root/rhel-guest-image-7.2-20151102.0.x86_64.qcow2
          - os_image:
              cloud: ospcloud
              name: rhel-guest
              container_format: bare
              disk_format: qcow2
              state: present
              filename: /root/rhel-guest-image-7.2-20151102.0.x86_64.qcow2
        EOF
        ```
    * `ansible-playbook /etc/openstack/osp_image.yml`
    * `ansible localhost -m os_auth -a cloud=ospcloud`
    * `ansible localhost -m os_user_facts -a cloud=ospcloud -v`
    * Create a new OSP flavor:
      ```
      cat << EOF > osp_flavor.yml
      - hosts: jumpbox
          tasks:
          - name: Create m2.small flavor
            os_nova_flavor:
              cloud: ospcloud
              state: present
              name: m2.small
              ram: 2048
              vcpus: 1
              disk: 10
        EOF
      ```
    * `ansible-playbook osp_flavor.yml`

## Step #2 - Production AWS Setup for Tower

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
        * Set the *Instance Filters* to `tag:owner=npoyant@redhat.com`.  Or, whatever your resource identifer is.
    * TODO: Put section here about adding the groups pointing to AWS Groups for reference later on in the Provision_AWS.yml playbook.
3. Set up credentials in order for Ansible Tower to communicate to 3 Tier Resources.
    * Acquire the GUID setup from OpenTLC.
    * `ssh` to your bastion machine:
      ```
      [laptop ]$ ssh -i ~/.ssh/id_rsa npoyant-redhat.com@bastion.9ce4.example.opentlc.com
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

## 3. Setup Workflow Template



