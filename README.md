<h1> Lynx <h1>

<h2> Purpose: </h2>
Lynx is a platform enabling CSE to provision automatically environments for research and training purposes.<br>
This document explains how Lynx is structured and how you can contribute to the project.

<h2> Summary </h2>

##### Table of Contents  

[Architecture](#Architecture)<br>
[Access](#Access)<br>
[Provisioning an environment](#Provisioning)<br>
[Deprovisioning an environment](#Deprovisioning)<br>
[Documentation](#Documentation)<br>
[Contributing](#Contributing)<br>
[Anatomy of Training Lab Workflow](#Anatomy)<br>
[AWX Key Tabs](#AWXKeyTabs)<br>
[Troubleshooting](#Troubleshooting)<br>
[Roadmap](#Roadmap)<br>

<h2> Architecture</h2>
<a name="Architecture"/>
AWX is provisionned from: https://github.com/tigera-cs/lynx<br>
<img src="https://github.com/tigera-cs/lynx-core/blob/master/img/lynxarchdiagram.png"><br>
If the AWX instnace is to fail, this is the directory which will be used to reprovision AWX.<br>
 - AWX 17.1.0<br>
 - Python 3.6.8<br>
 - Ansible 2.10.6<br>

<h2> Access</h2><a name="Access"/>
You access the provisionning tool through https://lynx.tigera.ca/<br>
Use you tigera google credentials to identify<br>
For non-registered user, please contact david@tigera.io or sebastien@tigera.io to be added.<br>
ssh access to provisioned environment is possible. Add your ssh public key <a href="https://github.com/tigera-cs/ansible-tigera-internal/blob/master/tigera/internal/roles/lynx_bastion/defaults/main.yaml">here</a> so you can ssh into Bastion host.<br>

<h2> Provisioning an environment</h2><a name="Provisioning"/>
Once logged in click on the burger menu and select Templates under Resources.<br>
Here you will find workflows that will provision environments.<br>
01 Single Lab Workflow:<br>
This template will provision a bastion host and a 3 node kubernetes cluster. This is the template currently used for customer foundation and advanced trainning labs.<br>
 - Click on the template you want to launch<br>
 - Click launch<br>
   - When choosing name, DO NOT use a hyphen ` - ` as part of the name of the instance.<br>
   - Read ? for more information on what the field requiered does.<br>
   - Select present to provision an environment<br>
   - Traininglabs github repo willl allow ssh access with registered ssh key. To register you key refer to the access section of this document.<br>
   - Review configuration and Launch.<br>
   - The same step as above will have to be completed to decomission the environment with the expection of slecting absent instead of present.<br>

<h2> Deprovisioning an environment:</h2><a name="Deprovisioning"/>
 - Seclect the same Job workflow template that was used to create the environement<br>
 - Launch and change setting from present to absent<br>
 - Enter same name and settings as during creation. If you forgot you can look at the job history as reminder.<br>

<h2> Documentation</h2><a name="Documentation"/>
Documentation can be added, notably for Advanced training.<br>
Directory for md file is to be uploaded <a href="https://github.com/tigera-cs/tigera-lab/tree/master/trainingworkbooks/Advanced%20Training/labs">here</a><br>
Image references can be uploaded in a clearly marked directory <a href="https://github.com/tigera-cs/tigera-lab/tree/master/trainingworkbooks/Advanced%20Training/img">here</a><br>

<h2> Contributing and creating your own Workflow Job Template:</h2><a name="Contributing"/>
 - Create your own branches under tigera-cs - ansible-tigera-internal.<br>
 - Duplicate an existing template and rename as yours<br>
 - Duplicate an existing Project and rename as yours. Edit it so type matches your branch<br>
 - Map to your inventory<br>
 - The Job, Inventory and Workflow templates that you develop and should be used in production should be added as automation <a href="https://github.com/tigera-cs/lynx/blob/main/roles/awx-configure/tasks/main.yaml">here</a><br>
 - Reference your branch under <a href=:"https://github.com/tigera-cs/lynx-core/blob/whisperish/collections/requirements.yaml"> here <a>

<h2> Anatomy of Training Lab Workflow or how to get started once you want to customize yout project</h2><a name="Anatomy"/>
A workflow job template - 01 Single Lab Workflow - is launched after applying the variables selected during the survey.
It will run a series of sequentials tasks.
1. Run the template Single EC2

  - The Lynx Core project ec2.yaml is [executed](https://github.com/tigera-cs/lynx-core/blob/master/ec2.yaml)
    - It will create the following [instances](https://github.com/tigera-cs/ansible-tigera-internal/blob/master/tigera/internal/roles/lynx_ec2/tasks/create.yaml)
      - [Bastion host](https://github.com/tigera-cs/ansible-tigera-internal/blob/master/tigera/internal/roles/lynx_ec2/files/terraform/bastion.tf) with ubuntu-20.04 by default
      - [Control host](https://github.com/tigera-cs/ansible-tigera-internal/blob/master/tigera/internal/roles/lynx_ec2/files/terraform/control.tf)
      - 2 x [Worker hosts](https://github.com/tigera-cs/ansible-tigera-internal/blob/master/tigera/internal/roles/lynx_ec2/files/terraform/worker.tf)
      - [VPC with subnet & AWS Security Groups](https://github.com/tigera-cs/ansible-tigera-internal/blob/master/tigera/internal/roles/lynx_ec2/files/terraform/vpc.tf)
    
    - and the following NLB [object](https://github.com/tigera-cs/ansible-tigera-internal/blob/master/tigera/internal/roles/lynx_ec2_nlb/tasks/create.yaml)
      - DNS
      - NLB
      - Associated Target groups
     
2. AWX inventory synchronisation is executed against source - apply to dynamic inv. mapped to cloud resource

3. The Lynx Core project lab.yaml is [executed](https://github.com/tigera-cs/lynx-core/blob/master/lab.yaml)

  - Wait for Bastion to be available and [configure Bastion](https://github.com/tigera-cs/ansible-tigera-internal/blob/master/tigera/internal/roles/lynx_bastion/tasks/main.yaml)

  - Configure Worker and Controller
    - Add Bastion ssh
    - Install [Docker](https://github.com/tigera-cs/ansible-tigera-internal/blob/master/tigera/internal/roles/docker/tasks/Ubuntu.yaml)
    - Install [Kubeadm](https://github.com/tigera-cs/ansible-tigera-internal/blob/master/tigera/internal/roles/kubeadm/tasks/Ubuntu.yaml) 
  
  - Configure Controller - Option for multiple Master not activated
    - Run [kubeadm init](https://github.com/tigera-cs/ansible-tigera-internal/blob/master/tigera/internal/roles/kubeadm/tasks/init.yaml)
    - Set & Install [kubeconfig](https://github.com/tigera-cs/ansible-tigera-internal/blob/master/tigera/internal/roles/kubeadm/tasks/kubeconfig.yaml)
  
  - Join Workers
    - Run [join](https://github.com/tigera-cs/ansible-tigera-internal/blob/master/tigera/internal/roles/kubeadm/tasks/join.yaml)
    - Install [kubeconfig](https://github.com/tigera-cs/ansible-tigera-internal/blob/master/tigera/internal/roles/kubeadm/tasks/kubeconfig.yaml)
  
  - Install [Kubeconfig on Bastion](https://github.com/tigera-cs/ansible-tigera-internal/blob/master/tigera/internal/roles/kubeadm/tasks/kubeconfig.yaml)
  
  - Deploy [NGINX Ingress Controller](https://github.com/tigera-cs/ansible-tigera-internal/blob/master/tigera/internal/roles/lynx_bastion/templates/ingresscontroller.yaml.j2)

<h2> AWX Key Tabs</h2><a name="AWXKeyTabs"/>

<h3>Templates:</h3>
 - Use to provision environment<br>
 - Workflow executed to run multiple job template that leads to the creation of the environment.<br>
   - Flow described under Visualizer tab<br>
   - Survey allows you to set presets when Workflow template is launched<br>
 - Job template link to a project that combines an Inventory & Project<br>


<h3>Inventory:</h3>
 - Inventory:<br>
It is a collection of hosts against which jobs may be launched, the same as an Ansible inventory file. Inventories are divided into groups and these groups contain the actual hosts. Groups may be sourced manually, by entering host names into Tower, or from one of Ansible Tower’s supported cloud providers.<br>
 - Source:<br>
 If you already have an inventory source set up, then Tower automatically switches to use the inventory plugins depending on the source and Ansible version, but continue to maintain the same content previously in those scripts. If you need to control the version of Ansible being used, you can use custom virtual environments for the inventory source.<br>

<h3>Projects:</h3>
Assosiate a github repo and a branch from which manifest will be run<br>


<h2> Troubleshooting</h2><a name="Troubleshooting"/>
 - Under templates, identify the job ID that failed.<br>
 - Go under Jobs, select search on ID, enter ID and search<br>
 - Select output and look if any red<br>
 - You can select each steps to drill down and go under ouptut to see if ansible ran as epxected or if some steps are missed/skipped.<br>

<h2> Roadmap</h2><a name="Roadmap"/>

- v1 
   - Add multi-lab provisioning<br>
   - Update AWX repo with core framework (templates, workflows)<br>
   - Single inventory multi-region<br>
   - Calico Enterprise license and pull-secret as AWX Credentials<br>
   - DNS Wildcard - Done

- v2
  - Openshift<br>
  - Azure as a Cloud Provider<br>

- Wishlist
  - Dynamic inventory should use the correct user depending on OS, i.e. ubuntu for ubuntu and ec2-user for RH<br>
