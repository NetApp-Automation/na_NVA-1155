### Automated Oracle RAC deployment with Ansible for NVA-1155: Oracle 19c RAC Databases on FlexPod Datacenter with Cisco UCS and NetApp A800 AFF over FC

Ansible is a declarative, state-based, idempotent configuration management platform. It uses simple to read/write YAML files to describe the configuration.  This repository contains Ansible roles and playbooks for building Oracle 19c RACcluster and create RAC database thereafter as per NVA-1155 deployment guide. This is offered as an optional automated solution deployment to complement manual deployment process as outlined in the deployment guide. The automated Oracle RAC deployment is built with and consists of folowing ansible roles:
	
	linux_config
        storage_config
        grid_install
        db_install
        create_database
        postinst_config

Each role contains a number of tasks that deliver the required Oracle installation and configuration. These roles are developed as per the best practices prescribed in NetApp Validated Architectures NVA-1155. It is built with avaiable Ansible modules at time for implementing Oracle RAC cluster and database deployment on FlexPod infrastructure. 

### Limitation:

As the automation solution is specifically build for NVA-1155. The current roles and playbooks support following components and configurations:

        Operating System: Oracle Linux 8.2 RHCK 4.18.0-193.el8.x86_64
        Storage System: ONTAP 9.7
        Storage Protocol: FC
        Oracle Version: 19c grid, 19.8RU, 19c EE database 19.8RU
        Oracle Storage Type: XFS for binary, ASM for DATA and REDO
        Oracle Instance Type: Oracle RAC

It should point out that although Ansible is idempotent, due to scarcity of available Oracle database deployment modules, some of Oracle installation and configuration functions are built on Ansible Command module, which are not idempotent.

### Prerequisite

Before implementing this automated RAC deployment, it is assumed that basic Oracle Linux 8.2 operating systems have been installed on bare metal FlexPod compute blades and boot into RHCK kernel as versu Orcle UEK. The required Oracle FC luns has been created on A800 storage aggregats and each Oracle RAC node has been granted access to its own lun for Oracle binary and shared DATA and REDO and CRS luns.  

### Getting Started

These deployment instructions will enable you to create a VM as an Ansible controller and download the specific solution playbook files. After reviewing and editing few parameter files that fits your environment configuraton and deloyment goals. The Ansible automation playbooks can be executed to deploy Oracle 19c RAC installation and configuration on FlexPod infrastructureusing as per NVA-1155.

This solution specific NetApp Ansible roles are located at [https://bitbucket.ngage.netapp.com/scm/ciac-bb/ansible-toolkit.git](https://bitbucket.ngage.netapp.com/scm/ciac-bb/ansible-toolkit.git) repository. So we will refer it as 
"NetApp Solution Repo" in this documentation.

First you need to start with a VM template to build Ansible control VM. Then get Ansible roles and playbook via git clone or dowwnload this repo to configure your Oracle 19c RAC cluster on FlexPod. 

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
       git clone -b oracle_deployoment https://bitbucket.ngage.netapp.com/projects/CIAC-BB/repos/ansible-toolkit.git

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
[ontap]
10.61.185.161

### Executing the playbook
For FlexPod deployment, there is a a800_flexpod_9.7.yml file that is the playbook at the top level that calls the required roles for A800 deployment. Once you are done revising with all three parameter files, use following command to kick off the playbook. Playbook is executed under a user id admin at Ansible control VM.

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

TBD

### Additional information
