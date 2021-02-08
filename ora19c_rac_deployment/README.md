### Automated Oracle RAC deployment with Ansible for NVA-1155: Oracle 19c RAC Databases on FlexPod Datacenter with Cisco UCS and NetApp A800 AFF over FC

Ansible is a declarative, state-based, idempotent configuration management platform. It uses simple to read/write YAML files to describe the configuration.  This repository contains Ansible roles and playbooks for building Oracle 19c RACcluster and create RAC database thereafter as per NVA-1155 deployment guide. This is offered as an optional automated solution deployment to complement manual deployment process as outlined in the deployment guide. The automated Oracle RAC deployment is built with and consists of folowing ansible roles:
	
	linux_config - kernal configuration, required packages, resources limit etc.
        storage_config - Setup Oracle binary lun, multipath config
        grid_install - Install 19c grid infrastructure, patched to 19.8
        db_install - Install 19c database EE software only, patched to 19.8
        create_database - Create 19c CDB and create three PDBs in each container DB
        postinst_config - Configure oracle user environment

Each role contains a number of tasks that deliver the required Oracle installation and configuration tasks as listed above. These roles are developed as per the best practices prescribed in NetApp Validated Architectures NVA-1155. It is built with avaiable Ansible modules at time for implementing Oracle 19c RAC cluster and 19c database deployment on FlexPod infrastructure. 

### Limitation:

As the automation solution is specifically build for NVA-1155. The current roles and playbooks support following components and configurations:

        Operating System: Oracle Linux 8.2 RHCK 4.18.0-193.el8.x86_64
        Storage System: ONTAP 9.7
        Storage Protocol: FC
        Oracle Version: 19c grid, 19.8RU, 19c EE database 19.8RU
        Oracle Storage Type: XFS for binary, ASM for DATA, REDO and CRS
        Oracle Instance Type: Oracle RAC

It should point out that although Ansible is idempotent, due to scarcity of available Oracle database deployment modules, some of Oracle installation and configuration functions are built on Ansible Command module, which are not idempotent.

### Prerequisite

Before implementing this automated RAC deployment, it is assumed that basic Oracle Linux 8.2 operating systems have been installed on bare metal FlexPod compute blades and boot into RHCK kernel as versu Orcle UEK. The required Oracle FC luns has been created on A800 storage aggregats and each Oracle RAC node has been granted access to its own lun for Oracle binary and shared DATA, REDO and CRS luns.  

### Getting Started

These deployment instructions will enable you to create a VM as an Ansible controller and download the specific solution roles, and playbook files. After reviewing and editing few parameter files that fits your environment configuraton and deloyment goals, the Ansible automation playbooks can be executed to deploy Oracle 19c RAC installation and configuration on FlexPod infrastructureusing as per NVA-1155.

This solution specific NetApp Ansible roles are located at [https://bitbucket.ngage.netapp.com/projects/NS-BB/repos/na_nva-1155.git](https://bitbucket.ngage.netapp.com/projects/NS-BB/repos/na_nva-1155.git) repository. So we will refer it as "NetApp Solution Repo" in this documentation.

First you need to start with a VM template to build Ansible control VM. Then get Ansible roles and playbook via git clone or dowwnload this repo to configure your Oracle 19c RAC cluster on FlexPod. 

### Ansible Control VM (instructions provided here are tested with CentOS 8.2)

1. Build a CentOS 8.2 VM
Typical CentOS 8.2 OS installation should be sufficient. Create a admin user id to own and execute the playbook from this Ansible controll VM. The VM needs to have networking interfaces and configuration to access A800 storage controller as well as Oracle RAC nodes or other commponents within the FlexPod environment. The VM also needs to have nework acess to NetApp Ansible automation solution respository.  

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
There are several parameter files that are driving Oracle 19c RAC deployment in vars, and host_var folders under ora19c_rac_deployment root directory as listed below:

 * oracle_default.yml 
 * oracle_main.yml
 * b200-ora-01.yml, b200-ora-02.yml, b200-ora-03.yml ... 

Oracle_default.yml includes parameters that normally apply to all environments and do not need to be changed under normal circumstances. Oracle_main.yml includes most environment specific parameters. Each RAC node has a node specific parameter file under host_var directory. Review the parameter files carefully and modified the parameters in host_vars and oracle_main.yml for your environment specific values such as IPs, FC lun IDs etc.

The oracle_main.yml parameter file and host_vars files are populated with values from solution development in NetApp lab. That most likely does not fit into your current environment. It is important that you review these prameter files and make changes as necessary.

###  Roles and main.yml files
The deployment tasks are organized under roles and maintained in main.yml file in tasks folder of each role. The tasks are highly parameterized and under normal circumstances, there is no need to change anything in roles foler with exception noted below. 

###  Setting up template files for storage configuration
For this Oracle 19c RAC deployment solution, it is necessary to make changes in two template files in roles/storage_config/multipath.template and roles/storage_config/udev.template because they are environment specific with inputs from storage lun IDs output. We are also use different udev and multipath.conf files for RAC deployment (with device alias) and SnapCenter for Oracle backup (without device alias). Therefore, please use different udev rule and multipath.conf file in each situation. This can be easily changed by calling specific deployment tasks. This workaround exists because SnapCenter at this moment does not support device alias naming. After RAC deployment, modification to udev config and multipath.conf to remove device alias names has no impact to Oracle RAC funcationality as validated. 

For retrieving Oracle storage lun IDs from A800 cluster, login to A800 cluster via command line and execute following commands and then popluate the multipath.template and udev.template accordingly.

  ```
   set -showseparator ","
   lun show -vserver ora19c_svm -fields serial-hex

  ```    
This is assuming that Oracle luns are created under ora19c_svm.

###  Setting up host inventory file
This is the host names or IP addresses for your UCS compute blades
e.g.
  ```
   [oracle]
   b200-ora-01
   b200-ora-02
   b200-ora-03
   b200-ora-04
   b200-ora-05
   b200-ora-06
   b200-ora-07
   b200-ora-08

  ```

### Executing the playbook
For FlexPod RAC deployment, they are several playbook files that are created at the top level that calls the required roles for RAC deployment. Once you are done revising with all required parameter and template files, use following command to kick off the playbooks. Playbook is executed under a user id admin at Ansible control VM and passwordless authentication to RAC nodes should have been configured during controller VM setup.

  ```
   ansible-playbook -i hosts ora19c_linux_config.yml // Linux kernal configuration and storage setup on the hosts // 

  ```
and 

  ```
   ansible-playbook -i hosts ora19c_grid_install.yml // Oracle grid infrastructure installation //

  ```
and
 
  ```
   ansible-playbook -i hosts ora19c_db_install.yml // Oracle 19c database EE software only installation //

  ``` 
and

  ```
   ansible-playbook -i hosts ora19c_create_db.yml // Create a container DB or CDB and three PDBs in CDB //

  ```
or

  ```
   ansible-playbook -i hosts ora19c_rac_flexpod.yml // Include all above in single execution //

  ```

Execution with -t xxx option following the main yml file above only execute a particular role or tasks as identified by the tag id. To create additional CDB and PDBs within the RAC cluster. Change oracle_sid variable and EMEXPRESS port and run ora19c_create_db.yml again.

### Authors

 * Allen Cao (allen.cao@netapp.com)
 
### License

Read LICENSE.TXT

### Additional information
