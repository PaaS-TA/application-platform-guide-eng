### [Index](https://github.com/PaaS-TA/Guide-eng/blob/master/README.md) > [AP Install](../README.md) > RabbitMQ Service

## Table of Contents

1. [Document Outline](#1)   
  1.1. [Purpose](#1.1)   
  1.2. [Range](#1.2)   
  1.3. [References](#1.3)   

2. [RabbitMQ Service Installation](#2)  
  2.1. [Prerequisite](#2.1)   
  2.2. [Stemcell Check](#2.2)    
  2.3. [Deployment Download](#2.3)   
  2.4. [Deployment File Modification](#2.4)  
  2.5. [Service Installation](#2.5)    
  2.6. [Service Installation Check](#2.6)  

3. [Description for Sample App including RabbitMQ Linkage](#3)  
  3.1. [Service Broker Registration](#3.1)  
  3.2. [Sample App Download](#3.2)  
  3.3. [Request for service in PaaS-TA](#3.3)  
  3.4. [Request for service bind to Sample App and check for App](#3.4)   
     
## <div id='1'> 1. Document Outline
### <div id='1.1'> 1.1. Purpose
This document (Rabbit MQ service pack installation guide) describes how to install RabbitMQ service pack, which is a service pack provided by PaaS-TA, using Bosh. 

### <div id='1.2'> 1.2. Range
The installation range was prepared based on the basic installation to verify the RabbitMQ service pack. 


### <div id='1.3'> 1.3. References
BOSH Document: [http://bosh.io](http://bosh.io)  
Cloud Foundry Document: [https://docs.cloudfoundry.org](https://docs.cloudfoundry.org)  


## <div id='2'> 2. RabbitMQ Service Installation

### <div id="2.1"/> 2.1. Prerequisite  

This installation guide is based on installing in a Linux environment. 
In order to install the service pack, BOSH CLI v2 must be installed and logged in to BOSH.
If BOSH CLI v2 is not installed, you should first refer to the BOSH 2.0 installation guide document to install BOSH CLI v2 and familiarize the usage.

- Check the bosh runtime-config to see if there is a rabbitmq in the bosh-dns include deployments.
 ※ (Note) If bosh-dns include deployments is not at rabbitmq, go to ~/workspace/paasta-deployment/bosh/runtime-configs and open dns.yml to add rabbitmq and update bosh runtime-config.  

> $ bosh -e micro-bosh runtime-config
```
Using environment '10.0.1.6' as client 'admin'

---
addons:
- include:
    deployments:
    - paasta
    - pinpoint
    - pinpoint-monitoring
    - rabbitmq
    stemcell:
    - os: ubuntu-trusty
    - os: ubuntu-xenial
    - os: ubuntu-bionic
  jobs:
  - name: bosh-dns
    properties:
      api:
        client:
          tls: "((/dns_api_client_tls))"
        server:
          tls: "((/dns_api_server_tls))"
      cache:
        enabled: true
      health:
        client:
          tls: "((/dns_healthcheck_client_tls))"
        enabled: true
        server:
          tls: "((/dns_healthcheck_server_tls))"
    release: bosh-dns
  name: bosh-dns
...(Skip)...

Succeeded
```

### <div id="2.2"/> 2.2. Stemcell Check

Check the Stemcell list to make sure that the Stemcell required for service installation is uploaded.
The Stemcell of this guide uses ubuntu-bionic 1.76.

> $ bosh -e ${BOSH_ENVIRONMENT} stemcells

```
Using environment '10.0.1.6' as client 'admin'

Name                                       Version   OS             CPI  CID  
bosh-openstack-kvm-ubuntu-bionic-go_agent  1.76      ubuntu-bionic  -    ce507ae4-aca6-4a6d-b7c7-220e3f4aaa7d

(*) Currently deployed

1 stemcells

Succeeded
```

If the corresponding Stemcell is not uploaded, copy the Stemcell link to the corresponding IaaS environment and version from [bosh.io Stemcell](https://bosh.io/stemcells/) and run the following command.

```
# Example of Stemcell Upload Command
$ bosh -e ${BOSH_ENVIRONMENT} upload-stemcell -n {STEMCELL_URL}
```

### <div id="2.3"/> 2.3. Deployment Download  

Download the deployment needed from Git Repository and place the file at the service installation directory 

- Service Deployment Git Repository URL : https://github.com/PaaS-TA/service-deployment/tree/v5.1.5

```
# Deployment File Download , make directory, change directory
$ mkdir -p ~/workspace
$ cd ~/workspace

# Deployment File Download
$ git clone https://github.com/PaaS-TA/service-deployment.git -b v5.1.5

# common_vars.yml File Download (Download if common_vars.yml doesn't exist)
$ git clone https://github.com/PaaS-TA/common.git
```

### <div id="2.4"/> 2.4. Deployment File Modification

The BOSH Deployment manifest is a YAML file that defines the properties of components elements and deployments. 
Cloud config is used for network, vm_type, and disk_type used in Deployment files, and refer to the PaaS-TA AP installation guide on the usage.  

- Check the Cloud config settings.  

> $ bosh -e micro-bosh cloud-config   

```
Using environment '10.0.1.6' as client 'admin'

azs:
- cloud_properties:
    availability_zone: ap-northeast-2a
  name: z1
- cloud_properties:
    availability_zone: ap-northeast-2a
  name: z2

... ((Skip)) ...

disk_types:
- disk_size: 1024
  name: default
- disk_size: 1024
  name: 1GB

... ((Skip)) ...

networks:
- name: default
  subnets:
  - az: z1
    cloud_properties:
      security_groups: paasta-security-group
      subnet: subnet-00000000000000000
    dns:
    - 8.8.8.8
    gateway: 10.0.1.1
    range: 10.0.1.0/24
    reserved:
    - 10.0.1.2 - 10.0.1.9
    static:
    - 10.0.1.10 - 10.0.1.120

... ((Skip)) ...

vm_types:
- cloud_properties:
    ephemeral_disk:
      size: 3000
      type: gp2
    instance_type: t2.small
  name: minimal
- cloud_properties:
    ephemeral_disk:
      size: 10000
      type: gp2
    instance_type: t2.small
  name: small

... ((Skip)) ...

Succeeded
```

- Modify common_vars.yml to suit the server environment. 
- The variables used in RabbitMQ are system_domain, paasta_admin_username, paasta_admin_password, and paasta_nats_ip.

> $ vi ~/workspace/common/common_vars.yml
```
... ((Skip)) ...

system_domain: "61.252.53.246.nip.io"		# Domain (Same as HAProxy Public IP when using nip.io)
paasta_admin_username: "admin"			# PaaS-TA Admin Username
paasta_admin_password: "admin"			# PaaS-TA Admin Password
paasta_nats_ip: "10.0.1.121"
  
... ((Skip)) ...

```


- Modify the variable files used by Deployment YAML to suit the server environment.

> $ vi ~/workspace/service-deployment/rabbitmq/vars.yml

```
deployment_name: "rabbitmq"                                  # rabbitmq deployment name 

# STEMCELL
stemcell_os: "ubuntu-bionic"                                # stemcell os
stemcell_version: "1.76"                                    # stemcell version

# VM_TYPE
vm_type_small: "minimal"                                    # vm type small 

# NETWORK
private_networks_name: "default"                            # private network name

# COMMON
bosh_name: "micro-bosh"                                     # bosh name (e.g. micro-bosh) -- (Checkable through 'bosh env' command)
paasta_deployment_name: "paasta"                            # paasta application platform name (e.g. paasta)

# RABBITMQ
rabbitmq_azs: [z3]                                          # rabbitmq : azs
rabbitmq_instances: 1                                       # rabbitmq : instances (1) 
rabbitmq_private_ips: "<RABBITMQ_PRIVATE_IPS>"              # rabbitmq : private ips (e.g. "10.0.81.31")
management_username: "<MANAGEMENT_USERNAME>"  		    # rabbitmq : username (e.g. "madmin") *broker/administrator_username != management_username

# HAPROXY
haproxy_azs: [z3]                                           # haproxy : azs
haproxy_instances: 1                                        # haproxy : instances (1) 
haproxy_private_ips: "<HAPROXY_PRIVATE_IPS>"                # haproxy : private ips (e.g. "10.0.81.32")

# SERVICE-BROKER
broker_azs: [z3]                                            # service-broker : azs
broker_instances: 1                                         # service-broker : instances (1)
broker_port: 4567                                           # service-broker : broker port (e.g. "4567")
broker_username: "<SERVICE_BROKER_USERNAME>"		    # service-broker : username (e.g. "admin") *broker/administrator_username != management_username
broker_password: "<SERVICE_BROKER_PASSWORD>"                # service-broker : password (e.g. "admin" no recommand)
administrator_username: "<SERVICE_BROKER_ADMIN_USERNAME>"   # servier-broker : administrator username (e.g. "administrator")

# BROKER-REGISTRAR
broker_registrar_azs: [z3]                                  # broker-registrar : azs
broker_registrar_instances: 1                               # broker-registrar : instances (1) 

# BROKER-DEREGISTRAR
broker_deregistrar_azs: [z3]                                # broker-deregistrar : azs
broker_deregistrar_instances: 1                             # broker-deregistrar : instances (1)

```

### <div id="2.5"/> 2.5. Service Installation

- Modify the VARIABLES settings in the Deploy script file to suit the server environment, and decide whether to add the option file.
     (Optinal) -o operations/cce.yml (Apply CCE when Installing)

> $ vi ~/workspace/service-deployment/rabbitmq/deploy.sh

```
#!/bin/bash

# VARIABLES
COMMON_VARS_PATH="<COMMON_VARS_FILE_PATH>"  # common_vars.yml File Path (e.g. ../../common/common_vars.yml)
BOSH_ENVIRONMENT="${BOSH_ENVIRONMENT}"      # bosh director alias name (WHen not useing create-bosh-login.sh provided by PaaS-TA, check the name in bosh envs and enter)

# DEPLOY
bosh -e ${BOSH_ENVIRONMENT} -n -d rabbitmq deploy --no-redact rabbitmq.yml \
    -o operations/cce.yml \
    -l ${COMMON_VARS_PATH} \
    -l vars.yml
```

- Service Installation 
```
$ cd ~/workspace/service-deployment/rabbitmq  
$ sh ./deploy.sh  
```  


### <div id="2.6"/> 2.6. Service Installation Check

Check the installed service.  

> $ bosh -e micro-bosh -d rabbitmq vms  

```
Using environment '10.30.40.111' as user 'admin' (openid, bosh.admin)

Task 8077. Done

Deployment 'rabbitmq'

Instance                                                Process State  AZ  IPs            VM CID                                   VM Type  Active  
haproxy/a30fb543-000d-4f74-b62d-7418da0e6101            running        z5  10.30.107.192  vm-fbd4a04a-5346-4e00-b793-17c327f90aa7  minimal  true  
rmq-broker/52629ddb-32c9-4097-b9f6-e5dc0aff55ce         running        z5  10.30.107.191  vm-5238f05b-ec4f-449c-ab1d-a1a5b932d76e  minimal  true  
rmq/a4ef4c7e-4776-411d-8317-b2b059e416dd                running        z5  10.30.107.193  vm-f8d8a62d-bfc4-442e-8306-9f133ebfc518  minimal  true  

3 vms

Succeeded
```

## <div id='3'> 3. Description for Sample App including RabbitMQ Linkage

This Sample App is deployed to PaaS-TA and can be used with RabbitMQ's service on Provision and Bind.

### <div id='3.1'> 3.1. Service Broker Registration

When the RabbitMQ service pack deployment is completed, you must first register the RabbitMQ service broker to use the service pack in the application.
When registering a service broker, you must log in as a user who can register a service broker in PaaS-TA.

- Check the list of service brokers.
> $ cf service-brokers
```
Getting service brokers as admin...
  
name   url
No service brokers found
```

<br>

- Command for registering Service Broker
```
cf create-service-broker [SERVICE_BROKER] [USERNAME] [PASSWORD] [SERVICE_BROKER_URL]

[SERVICE_BROKER] : Service Broker Name
[USERNAME] / [PASSWORD] : User ID / PASSWORD with access to service broker
[SERVICE_BROKER_URL] : Service Broker Access URL
```
	
- Register RabbitMQ service broker.

> $ cf create-service-broker rabbitmq-service-broker admin cloudfoundry http://<rmq-broker_ip>:4567
```
$ cf create-service-broker rabbitmq-service-broker admin cloudfoundry http://10.30.107.191:4567
Creating service broker rabbitmq-service-broker as admin...
OK

```
<br>

- Check the registered RabbitMQ service broker.

> $ cf service-brokers  

```
Getting service brokers as admin...

name                    url
rabbitmq-service-broker http://10.30.107.191:4567
```
<br>

- Check the list of accessible services.

> $ cf service-access

```
Getting service access as admin...
broker: rabbitmq-service-broker
   service      plan       access   orgs
   rabbitmq     standard   none      
```

- Access is initially not permitted when registering as a service broker. Therefore, access is set to none.

- Assign permission to a specific organization to access the service and recheck the access service list. (Overall Organization)

> $ cf enable-service-access rabbitmq 

```
Enabling access to all plans of service rabbitmq for all orgs as admin...
OK
```

> $ cf service-access
```
Getting service access as admin...
broker: rabbitmq-service-broker
   service      plan       access   orgs
   rabbitmq     standard   all      
```

### <div id='3.2'> 3.2. Sample App Download

- Download Zip file of Sample Apps
```
$ wget https://nextcloud.paas-ta.org/index.php/s/NDgriPk5cgeLMfG/download --content-disposition  
$ unzip paasta-service-samples.zip  
$ cd paasta-service-samples/rabbitmq  
```

<br>


### <div id='3.3'> 3.3. Request for service
Request for service in order to use the RabbitMQ service in the Sample App, you must request for service (Provision).
*Note: When requesting for a service, you must be logged in as a user who can request for a service in PaaS-TA.

- Check whether there is a service in the PaaS-TA Marketplace first.

> $ cf marketplace

```
getting services from marketplace in org system / space dev as admin...
OK

service      plans         description                                                                           broker
rabbitmq     standard      RabbitMQ is a robust and scalable high-performance multi-protocol messaging broker.   rabbitmq-service-broker

TIP: Use 'cf marketplace -s SERVICE' to view descriptions of individual plans of a given service.
```
<br>

- Command for Service Instance Request
```
cf create-service [SERVICE] [PLAN] [SERVICE_INSTANCE]

[SERVICE] : Service name shown in the Marketplace
[PLAN] : Policies for Services
[SERVICE_INSTANCE] : Name of the service instance to create
```

- If there is a service you want on the Marketplace, request for a service (Provision).

> $ cf create-service rabbitmq standard my_rabbitmq_service
```
Creating service instance my_rabbitmq_service in org system / space dev as admin...
OK
```

<br>

- Check the generated instance of the RabbitMQ service.

> $ cf services

```
Getting services in org system / space dev as admin...

name                  service      plan       bound apps   last operation     broker                    upgrade available
my_rabbitmq_service   rabbitmq     standard                create succeeded   rabbitmq-service-broker   
```

<br>

### <div id='3.4'> 3.4. Request for service bind to Sample App and check App
Once the service request is completed, download the labbit-example-app provided by cf and conduct the test.
* Note: When requesting for service bind, you must be logged in as a user who can request for service bind in PaaS-TA..

- Check the manifest file.  

> $ vi manifest.yml   

```
---
applications:
- name: rabbit-example-app
  command: thin -R config.ru start
  path: .
  buildpacks:
  - ruby_buildpack
```

- Deploy app with --no-start option.

> $ cf push --no-start 
```  
Applying manifest file /home/ubuntu/workspace/samples/paasta-service-samples/rabbitmq/manifest.yml...
Manifest applied
Packaging files to upload...
Uploading files...
 3.16 MiB / 3.16 MiB [===================================================================================================

Waiting for API to complete processing files...

name:              rabbit-example-app
requested state:   stopped
routes:            rabbit-example-app.paasta.kr
last uploaded:     
stack:             
buildpacks:        

type:            web
sidecars:        
instances:       0/1
memory usage:    1024M
start command:   thin -R config.ru start
     state   since                  cpu    memory   disk     details
#0   down    2021-11-22T05:32:24Z   0.0%   0 of 0   0 of 0   

```  
  
- Request for service instance bind created by Sample Web App.

> $ cf bind-service rabbit-example-app my_rabbitmq_service 

```	
Binding service my_rabbitmq_service to app rabbit-example-app in org system / space dev as admin...
OK
```

When running the app, add a security group for communication with the service.

- Modify rule.json.  
> $ vi rule.json   
```
## Set haphroxy IP of rabbitmq to destination
[
  {
    "protocol": "all",
    "destination": "<haproxy_ip>"
  }
]
```
  
- Create a security group.  

> $ cf create-security-group rabbitmq rule.json

```
Creating security group rabbitmq as admin...

OK		
```
  
- Request the security group that you created to use the rabbitmq service.
> $ cf bind-running-security-group rabbitmq 
```
Binding security group rabbitmq to running as admin...
OK		
```
  
- Restart App for bind to take effect.

> $ cf restart rabbit-example-app 

```	
Restarting app rabbit-example-app in org system / space dev as admin...

Staging app and tracing logs...
   Downloading ruby_buildpack...
   Downloaded ruby_buildpack (5.2M)
   Cell 4a88ce8b-1e72-485a-8f62-1fe0c6b9a7cd creating container for instance 934daf45-8787-4d8a-86fd-bd6dbde78f30
   Cell 4a88ce8b-1e72-485a-8f62-1fe0c6b9a7cd successfully created container for instance 934daf45-8787-4d8a-86fd-bd6dbde7
   Downloading app package...
   Downloaded app package (3.2M)
   -----> Ruby Buildpack version 1.8.37


........
........
Instances starting...
Instances starting...

name:              rabbit-example-app
requested state:   started
routes:            rabbit-example-app.paasta.kr
last uploaded:     Mon 22 Nov 05:34:27 UTC 2021
stack:             cflinuxfs3
buildpacks:        
	name             version   detect output   buildpack name
	ruby_buildpack   1.8.37    ruby            ruby

type:           web
sidecars:       
instances:      1/1
memory usage:   1024M
     state     since                  cpu    memory   disk     details
#0   running   2021-11-22T05:34:36Z   0.0%   0 of 0   0 of 0   
```  


-  Check if the app uses RabbitMQ service normally.


- Check at the browser

>![rabbitmq_image_12]

- Test Store Endpoints
```	
$ curl -XPOST -d 'test' https://rabbit-example-app.<YOUR-DOMAIN>/store -k  
$ curl -XGET https://rabbit-example-app.<YOUR-DOMAIN>/store -k  
test
```








[rabbitmq_image_01]:./images/rabbitmq/rabbitmq_image_01.png

[rabbitmq_image_12]:./images/rabbitmq/rabbitmq_image_12.png
[rabbitmq_image_13]:./images/rabbitmq/rabbitmq_image_13.png
[rabbitmq_image_14]:./images/rabbitmq/rabbitmq_image_14.png
[rabbitmq_image_15]:./images/rabbitmq/rabbitmq_image_15.png
[rabbitmq_image_16]:./images/rabbitmq/rabbitmq_image_16.png
[rabbitmq_image_17]:./images/rabbitmq/rabbitmq_image_17.png
[rabbitmq_image_18]:./images/rabbitmq/rabbitmq_image_18.png
[rabbitmq_image_19]:./images/rabbitmq/rabbitmq_image_19.png


### [Index](https://github.com/PaaS-TA/Guide-eng/blob/master/README.md) > [AP Install](../README.md) > RabbitMQ Service
