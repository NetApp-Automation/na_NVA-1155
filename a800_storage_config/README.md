### Automated ONTAP A800 Storage Deployment with Ansible for NVA-1155: Oracle 19c RAC Databases on FlexPod Datacenter with Cisco UCS and NetApp A800 AFF over FC

Ansible is a declarative, state-based, idempotent configuration management platform. It uses simple to read/write YAML files to describe the configuration.  This repository contains Ansible roles and playbooks for an end-to-end storage deployment on A800 HA controller pair for Oracle 19c RAC database deployment as per NVA-1155 deployment guide. This is offered as an optional automated solution deployment to complement manual deployment process as outlined in the deployment guide. The A800 storage deployment automation is roles based and is built with folowing roles:
	
	primary_setup - General configuration of A800 controller pair
	ontap_svms - Create SVMs for infrastructure and Oracle 
	ontap_lifs - Create lifs for storage access
	ontap_igroups - Create igroup for each RAC node
	ontap_luns - Create Oracle luns based on predefined Oracle storage layout 

These roles are developed as per the best practices prescribed in NetApp Validated Architectures NVA-1155. It is built with avaiable Ansible modules at time for implementing storage deployment functions included in these roles for OracleRAC database deployment on FlexPod. 

### Limitation:

As the automation solution is specifically build for NVA-1155. The current roles and playbooks support following components and configurations:

        Operating System: Oracle Linux 8.2 RHCK 4.18.0-193.el8.x86_64
        Storage System: ONTAP 9.7
        Storage Protocol: FC
        Oracle Version: 19c grid with 19.8 patch, 19c Oracle database EE with 19.8 patch
        Oracle Storage Type: XFS for Oracle binary lun, ASM for shared DATA, REDO and CRS
        Oracle Instance Type: Oracle RAC

### Prerequisite

It is assumed that pre-setup activity has been completed on A800 HA controller pair including rack and stack, power-on to setup management IPs and initial ONTAP cluster building. The storage aggregats may or may not be created during cluster creation. If not, there is specific task within primary_setup role to create the aggregates as needed. 

### Getting Started

These deployment instructions will enable you to create VM from a template as an Ansible controller and download the specific solution playbook and run the Ansible roles to deploy storage configuration on A800 controller for Oracle 19c RAC database deployment that fits your specific envrironment using few parameter files.

This solution specific NetApp Ansible roles are located at [https://bitbucket.ngage.netapp.com/projects/NS-BB/repos/na_nva-1155.git](https://bitbucket.ngage.netapp.com/projects/NS-BB/repos/na_nva-1155.git) repository. So we will refer it as "NetApp Solution Repo" in this documentation.

First you need to start with a VM template to build Ansible control VM. Then get Ansible roles and playbook to configure your A800 storage system. 

### Ansible Control VM (instructions provided here are tested with CentOS 8.2)

1. Build a CentOS 8.2 VM
Typical CentOS 8.2 OS installation should be sufficient. Create a admin user id to to own and execute the playbook from this Ansible controll VM. The VM needs to have networking interfaces and configuration to access A800 storage controller as well as Oracle RAC nodes or other commponents within the FlexPod environment. The VM also needs to have nework acess to NetApp Ansible automation solution respository.  

2. Configure Ansible controller VM:
Following instructions are for CentOS / RHEL. For other OS, follow equivalent steps.

 * Install python3 
   [admin@ansiblectl ~]$ sudo dnf install python3

 * Installing PIP – The Python Package Installer 
   [admin@ansiblectl ~]$ sudo dnf install python3-pip

 * Installing Ansible
   [admin@ansiblectl ~]$ pip3 install ansible --user //This will install ansible version 2.10.3//

 * Generate ssh key for password less authentification 
   [admin@ansiblectl ~]$ ssh-keygen

 * Copy ssh id to Oracle hosts
   [admin@ansiblectl ~]$ ssh-copy-id admin@b200-ora-01 //This is not needed for A800 configuratioan but useful to configure Oracle RAC nodes//

 * Install NetApp library and collections
   [admin@ansiblectl ~]$ pip3 install netapp-lib solidfire-sdk-python requests --user
   [admin@ansiblectl ~]$ ansible-galaxy collection install netapp.ontap netapp.elementsw

 * Checking connectivity from VM to A800 controller cluster virtual IP and Oracle hosts ip as needed

3. Obtain NetApp Ansible automation roles and playbook from "NetApp Solution Automation repo":

Navigate to the NetApp Solution Automation repo: 

 * Download or clone solution roles in admin home directory. You would need credentials (or token) to access Git repo from ngage.netapp.com. 
 * or Run git clone

  ```
       git clone https://bitbucket.ngage.netapp.com/projects/NS-BB/repos/na_nva-1155.git

  ```
 
###  Setting up parameter files 
There are three parameter files that are driving A800 configuration in vars folder under ansible storage config root directory as listed below:

 * ontap_default.yml 
 * ontap_main.yml
 * ontap_lun_layout.yml

Ontap_default includes parameters that normally apply to all environments and do not need to be changed under normal circumstances. Ontap_main includes most environment specific parameters. Ontap_lun_layout is an example Oracle lun storage layout in A800 storage controller as we demonstrated in NVA-1155 solution. This can be modified for desired storage configration for Oracle workload to be deployed.

The ontap_main.yml parameter file is populated with values from solution development in NetApp lab. That most likely does not fit into your current environment. It is important that you review this prameter file and make changes as necessary.

###  Setting up host inventory file
This is the cluster host name or IP address for your ONTAP A800 cluster
e.g.
  ```
  [ontap]
  10.61.184.192

  ```
### Executing the playbook
For FlexPod deployment, we have created an playbook a800_flexpod_9.7.yml file at the top level that calls all required roles for A800 deployment. Once you are done revising with all three parameter files, use following command to kick off the playbook. Playbook is executed under a user id admin at Ansible control VM. You could also create your own playbook to call individual roles if so desired.

  ```
   ansible-playbook -i hosts a800_flexpod_9.7.yml 

  ```
or 

  ```
   ansible-playbook -i hosts a800_flexpod_9.7.yml -t xxx

  ```

Execution with -t option only execute a particular role or tasks as identified by the tag id.

### Authors

 * Allen Cao (allen.cao@netapp.com)
 
### License

Read LICENSE.TXT

### Additional information

