# Fuel Plugin ScaleIO on Cinder for OpenStack

### Contents

- [Introduction](#Introduction)
- [Requirements](#requirements)
- [Limitations](#limitations)
- [Configuration](#configuration)
	- [ScaleIO Cinder plugin installation](#scaleio-cinder-plugin-installation)
	- [ScaleIO Cinder plugin configuration](#scaleio-cinder-plugin-configuration)
	- [ScaleIO Cinder plugin operations](#scaleio-cinder-plugin-operations)
- [Contributions](#contributions)
- [License](#licensing)


## Introduction

Fuel plugin for ScaleIO for enabling OpenStack to work with an **External** ScaleIO deployment. This ScaleIO plugin for Fuel extends Mirantis OpenStack functionality by adding support for ScaleIO block storage.

ScaleIO is a software-only solution that uses existing servers' local disks and LAN to create a virtual SAN that has all the benefits of external storage—but at a fraction of cost and complexity. ScaleIO utilizes the existing local internal storage and turns it into internal shared block storage.

The following diagram shows the plugin's high level architecture: 

![ScaleIO Fuel plugin high level architecture](https://github.com/emccode/fuel-plugin-scaleio-cinder-test/blob/master/documentation/images/fuel-plugin-scaleio-cinder-1.png)


From the figure we can see that we need the following OpenStack roles and services: 


Service/Role Name | Description | Installed in |
|------------|-------------|--------------|
|Controller Node + Cinder Host |A node that runs network, volume, API, scheduler, and image services. Each service may be broken out into separate nodes for scalability or availability. In addition this node is a Cinder Host, that contains the Cinder Volume Manager|OpenStack Cluster|
|Compute Node |A node that runs the nova-compute daemon that manages Virtual Machine (VM) instances that provide a wide range of services, such as web applications and analytics.|OpenStack Cluster|


In the **external ScaleIO cluster** we have installed the following roles and services: 

Service Name | Description | Installed in |
|------------|-------------|--------------|
|SclaeIO Gateway (REST API)|The ScaleIO Gateway Service, includes the REST API to communicate storage commands to the SclaeIO Cluster, in addtion this service is used for authentication and certificate management.|ScaleIO Cluster|
|Meta-data Manager (MDM)|Configures and monitors the ScaleIO system. The MDM can be configured in redundant Cluster Mode, with three members on three servers, or in Single Mode on a single server.|ScaleIO Cluster|
|Tie Breaker (TB)|Tie Breaker service helps determining what service runs as a master vs. a slave |ScaleIO Cluster|
|Storage Data Server (SDS)|Manages the capacity of a single server and acts as a back-end for data access.The SDS is installed on all servers contributing storage devices to the ScaleIO system. These devices are accessed through the SDS.|ScaleIO Cluster| 
|Storage Data Client (SDC)|A lightweight device driver that exposes ScaleIO volumes as block devices to the application that resides on the same server on which the SDC is installed.|Openstack Cluster|

**Note:** for more information in how to deploy a ScaleIO Cluster, please refer to the ScaleIO manuals located in the download packages for your platform: [http://www.emc.com/products-solutions/trial-software-download/scaleio.htm](http://www.emc.com/products-solutions/trial-software-download/scaleio.htm "Download ScaleIO") and/or [watch the demo](https://community.emc.com/docs/DOC-45019 "Watch our demo to learn how to download, install, and configure ScaleIO")



## Requirements

These are the plugin requirements: 


| Requirement                                              | Version/Comment |
|----------------------------------------------------------|-----------------|
| Mirantis OpenStack compatibility                         | >= 6.1          |
| ScaleIO Version										   | >= 1.32         |
| Controller and Compute Nodes' Operative System		   | CentOS/RHEL 6.5 |
| OpenStack Cluster (Controller/cinder-volume node) can access ScaleIO Cluster | via a TCP/IP Network  |
| OpenStack Cluster (Compute nodes) can access ScaleIO Cluster| via a TCP/IP Network  |
| Install ScaleIO Storage Data Client (SDC) in Controller and Compute Nodes| Plugin takes care of install|


## Limitations

Currently Fuel doesn't support multi-backend storage.


## Configuration


Plugin files and directories:

|File/Directory|Description|
|--------------|-----------|
|Deployment_scripts| Folder that includes the bash/puppet manifests for deploying the services and roles required by the plugin|
|Deployment_scripts/puppet||
|environment_config.yaml|Contains the ScaleIO plugin parameters/fields for the Fuel web UI|
|metadata.yaml|Contains the name, version and compatibility information for the ScaleIO plugin|
|pre_build_hook|Mandatory file - blank for the ScaleIO plugin|
|repositories/centos|Empty Directory, the plugin scripts will download the required CentOS packages|
|repositories/Ubuntu|Empty Directory, not used|
|taks.yaml|Contains the information about what scripts to run and how to run them|


This Fuel plugin will install the ScaleIO Storage Data Client (SDC) service on each Controller node and Compute node in the cluster. This is necessary in order for the VMs in each compute node to utilize ScaleIO Storage:

![Plugin Architecture ](https://github.com/emccode/fuel-plugin-scaleio-cinder-test/blob/master/documentation/images/fuel-plugin-scaleio-cinder-2.png)


Before starting a deployment there are some things that you should verify:

1. Your ScaleIO Cluster can route 10G Storage Network to all Compute nodes
   as well as the Cinder Control/Manager node.
2. Create an account on the ScaleIO cluster to use as the OpenStack Administrator
   account (use the login/password for this account as san_login/password settings).
3. Obtain the IP address from the ScaleIO cluster

### ScaleIO Cinder plugin installation

The first step is to install the ScaleIO Cinder plugin in the Fuel Master:

1. Download the plugin from the releases page in the Fuel Master directory: [https://github.com/emccode/fuel-plugin-scaleio-cinder-test/releases ](https://github.com/emccode/fuel-plugin-scaleio-cinder-test/releases  "Releases' page")
2. Install the plugin in the fuel master using the following command: `installation command goes here`
3. Verify that the plugin has been installed successfully: 

![Plugin Installation](https://github.com/emccode/fuel-plugin-scaleio-cinder-test/blob/master/documentation/images/scaleio-cinder-install-1.png)



### ScaleIO Cinder plugin configuration

Once the plugin has been installed in the Master, we configure the nodes and set the parameters for the plugin: 

1. Define the Roles for each one of the Nodes
![OpenStack Node configuration](https://github.com/emccode/fuel-plugin-scaleio-cinder-test/blob/master/documentation/images/scaleio-cinder-install-2.png)

2. Setup the ScaleIO Cluster parameters in the plugin UI
![ScaleIO Cluster Parameters](https://github.com/emccode/fuel-plugin-scaleio-cinder-test/blob/master/documentation/images/scaleio-cinder-install-3.jpg)

**Plugin's parameters explanation:** 

|Parameter Name|Parameter Description|
|--------------|---------------------|
|ScaleIO Repo URL| The URL of the ScaleIO sources repository. This is the URL for the required scaleIO zip file that contains the ScaleIO product. For our example we are using the URI for the [ScaleIO Linux download](http://downloads.emc.com/emc-com/usa/ScaleIO/ScaleIO_Linux_SW_Download.zip "ScaleIO Linux Download") located in the ScaleIO trial download at [EMC.com](http://www.emc.com/products-solutions/trial-software-download/scaleio.htm "ScaleIO Trial Download")|
|userName|The ScaleIO User Name|
|Password|The SclaeIO password for the selected user name|
|ScaleIO GW IP|The IP address of the the ScaleIO Gateway service|
|ScaleIO Primary IP|The ScaleIO cluster's primary IP address|
|ScaleIO Secondary IP|The ScaleIO cluster's secondary IP address|
|ScaleIO protection domain|Name of the ScaleIO's protection domain|
|ScaleIO storage pool 1|Name of the first storage pool|
|Driver Labels for the first storage Pool|List of driver labels for the first storage pool (comma separated)|
|ScaleIO storage pool 2|Name of the second storage pool (Optional)|
|Driver Labels for the second storage Pool|List of driver labels for the second storage pool (comma separated)|
|Driver Labels for the first storage Pool|List of driver labels for the first storage pool (comma separated)|
|Fault sets list|List of the fault sets (comma separated)|

**Note:** Please refer to the ScaleIO documentation for more information on these parameters 


3. Execute the Installation: 
![OpenStack Deployment Successful](https://github.com/emccode/fuel-plugin-scaleio-cinder-test/blob/master/documentation/images/scaleio-cinder-install-4.jpg)

### ScaleIO Cinder plugin operations

**[TODO]**




## Contributions

The Fuel plugin for ScaleIO project has been licensed under the  [Apache 2.0](http://www.apache.org/licenses/LICENSE-2.0") License. In order to contribute to the  project you will to do two things:


1. License your contribution under the [DCO](http://elinux.org/Developer_Certificate_Of_Origin "Developer Certificate of Origin") + [Apache 2.0](http://www.apache.org/licenses/LICENSE-2.0")
2. Identify the type of contribution in the commit message


### 1. Licensing your Contribution:

As part of the contribution, in the code comments (or license file) associated with the contribution must include the following:

Copyright [yyyy] [name of copyright owner]

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

This code is provided under the Developer Certificate of Origin- [Insert Name], [Date (e.g., 1/1/15]”


**For example:**

A contribution from **Joe Developer**, an **independent developer**, submitted in **May 15th of 2015** should have an associated license (as file or/and code comments) like this:

Copyright (c) 2015, Joe Developer

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

This code is provided under the Developer Certificate of Origin- Joe Developer, May 15th 2015”

### 2. Identifying the Type of Contribution

In addition to identifying an open source license in the documentation, **all Git Commit messages** associated with a contribution must identify the type of contribution (i.e., Bug Fix, Patch, Script, Enhancement, Tool Creation, or Other).


## Licensing

The fuel plugin for ScaleIO is licensed under the  [Apache 2.0](http://www.apache.org/licenses/LICENSE-2.0") license

Copyright (c) 2015, EMC Corporation

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.


## Support

Please file bugs and issues at the Github issues page. For more general discussions you can contact the EMC Code team at <a href="https://groups.google.com/forum/#!forum/emccode-users">Google Groups</a> or tagged with **EMC** on <a href="https://stackoverflow.com">Stackoverflow.com</a>. The code and documentation are released with no warranties or SLAs and are intended to be supported through a community driven process.
