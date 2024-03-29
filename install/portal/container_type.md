### [Index](https://github.com/PaaS-TA/Guide-eng/blob/master/README.md) > [AP Install](../README.md) > Portal Container Type

## Table of Contents  

1. [Document Outline](#1)  
    1.1. [Purpose](#1.1)  
    1.2. [Range](#1.2)  
    1.3. [References](#1.3)  

2. [PaaS-TA AP Portal infra Installation](#2)  
    2.1. [Prerequisite](#2.1)   
    2.2. [Stemcell Check](#2.2)    
    2.3. [Deployment Download](#2.3)   
    2.4. [Deployment File Modification](#2.4)  
    2.5. [Service Installation](#2.5)    
    2.6. [Service Installation Check](#2.6)  

3. [PaaS-TA AP Portal Installation](#3)  
    3.1. [Portal App Configuration](#3.1)  
    3.2. [Portal Set App Deployment Script Variables](#3.2)  
    3.3. [Portal Run App Deployment Script](#3.3)  

4. [PaaS-TA AP Portal Operation](#4)  
    4.1. [Enable the user's organizational creation flag](#4.1)  
    4.2. [User Portal UAA Page Error](#4.2)  
    4.3. [Apply Catalog](#4.3)  


## <div id="1"/> 1. Document Outline
### <div id="1.1"/> 1.1. Purpose

This document (PaaS-TA AP Portal Container Type Installation Guide) describes how to install PaaS-TA AP Portal using BOSH and PaaS-TA AP.

### <div id="1.2"/> 1.2. Range
The installation range was created based on the installation of the Portal infrastructure and the distribution of the Portal App to verify the PaaS-TA AP Portal.

### <div id="1.3"/> 1.3. References
BOSH Document: [http://bosh.io](http://bosh.io)  
Cloud Foundry Document: [https://docs.cloudfoundry.org](https://docs.cloudfoundry.org)  

## <div id="2"/> 2. PaaS-TA AP Portal infra Installation  

### <div id="2.1"/> 2.1. Prerequisite
This installation guide is based on installation in a Linux environment.
To install the service pack, BOSH CLI v2 must be installed and logged in to BOSH.
If BOSH CLI v2 is not installed, you must first refer to the BOSH 2.0 Installation Guide document to install BOSH CLI v2 and familiarize yourself with the usage.
If the UAA client is not installed, the UAA client needs to be installed.

- UAA client Installation (BOSH Dependency Installation Required)
```
$ sudo gem install cf-uaac
$ uaac -v
```

### <div id="2.2"/> 2.2. Stemcell Check
Check the list of Stemcells to verify that the Stemcells required for service installation are uploaded.
The Stemcell in this guide uses ubuntu-bionic 1.76.

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

- Portal Deployment Git Repository URL : https://github.com/PaaS-TA/portal-deployment/tree/v5.2.5

```
# Deployment File Download , make directory, change directory
$ mkdir -p ~/workspace
$ cd ~/workspace

# Deployment File Download
$ git clone https://github.com/PaaS-TA/portal-deployment.git -b v5.2.5
```

### <div id="2.4"/> 2.4. Deployment File Modification
The BOSH Deployment manifest is a YAML file that defines the properties of the Components element and the deployment. 
Network, vm_type, disk_type, etc. used in the deployment file utilize Cloud config, and refer to the PaaS-TA AP installation guide for utilization methods.

- Check the contents of the cloud config setting.  

> $ bosh -e ${BOSH_ENVIRONMENT} cloud-config   

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

- Modify common_vars.yml to suit your server environment.
- The variable used by PaaS-TA AP Portal infrastructure is system_domain.

> $ vi ~/workspace/common/common_vars.yml
```
... ((Skip)) ...

system_domain: "61.252.53.246.nip.io"		# Domain (Same as HAProxy Public IP when using nip.io)

... ((Skip)) ...
```


- Modify the variable files used by Deployment YAML to suit your server environment.

> $ vi ~/workspace/portal-deployment/portal-container-infra/vars.yml

```
# STEMCELL INFO
stemcell_os: "ubuntu-bionic"                                    # stemcell os
stemcell_version: "1.76"                                        # stemcell version

# NETWORKS INFO
private_networks_name: "default"                                # private network name

# PORTAL-INFRA INFO
infra_azs: [z3]                                                 # infra : azs
infra_instances: 1                                              # infra : instances (1)
infra_vm_type: "large"                                          # infra : vm type
infra_persistent_disk_type: "20GB"                              # infra : persistent disk type


# MARIADB INFO
mariadb_port: "<MARIADB_PORT>"                                  # mariadb : database port (e.g. 13306) -- Do Not Use "3306"
mariadb_admin_password: "<MARIADB_ADMIN_PASSWORD>"              # mariadb : database admin password (e.g. "Paasta@2019")
portal_default_api_name: "PaaS-TA"                              # portal default api name (e.g. PaaS-TA {Version})
portal_default_api_url: "http://<PORTAL_GATEWAY_ROUTE>"         # portal default api url (portal gateway url) (e.g. "http://portal-gateway.<DOMAIN>")
portal_default_header_auth: "Basic YWRtaW46b3BlbnBhYXN0YQ=="    # portal default header auth
portal_default_api_desc: "PaaS-TA infra"                        # portal default api description (e.g. PaaS-TA {Version} infra)


# BINARY_STORAGE INFO
binary_storage_auth_port: "<BINARY_STORAGE_AUTH_PORT>"          # binary storage : keystone port (e.g. 15001) -- Do Not Use "5000"
binary_storage_username: "<BINARY_STORAGE_USERNAME>"            # binary storage : username (e.g. "paasta-portal")
binary_storage_password: "<BINARY_STORAGE_PASSWORD>"            # binary storage : password (e.g. "paasta")
binary_storage_tenantname: "<BINARY_STORAGE_TENANTNAME>"        # binary storage : tenantname (e.g. "paasta-portal")
binary_storage_email: "<BINARY_STORAGE_EMAIL>"                  # binary storage : email (e.g. "paasta@paasta.com")
```

### <div id="2.5"/> 2.5. Service Installation

- Modify the VARIABLES settings in the Deploy script file to match your server environment, and decide whether to add the option file.
     (Optional) -o operations/cce.yml (Apply CCE when installing)

> $ vi ~/workspace/portal-deployment/portal-container-infra/deploy.sh
```
#!/bin/bash

# VARIABLES
COMMON_VARS_PATH="<COMMON_VARS_FILE_PATH>"  # common_vars.yml File Path (e.g. ../../common/common_vars.yml)
BOSH_ENVIRONMENT="${BOSH_ENVIRONMENT}"      # bosh director alias name (When not using create-bosh-login.sh provided by PaaS-TA, check the name at bosh envs and enter)

# DEPLOY
bosh -e ${BOSH_ENVIRONMENT} -n -d portal-container-infra deploy --no-redact portal-container-infra.yml \
   -o operations/cce.yml \
   -l ${COMMON_VARS_PATH} \
   -l vars.yml
```

- Install Service  
```
$ cd ~/workspace/portal-deployment/portal-container-infra    
$ sh ./deploy.sh  
```


### <div id="2.6"/> 2.6. Service Installation Check
Check the installed service.

> $ bosh -e ${BOSH_ENVIRONMENT} -d portal-container-infra vms  

```
Using environment '10.0.1.6' as client 'admin'

Task 1246. Done

Deployment 'portal-container-infra'

Instance                                    Process State  AZ  IPs          VM CID               VM Type  Active  
infra/3193a1fa-156d-4dd9-935d-4b67cdcc1182  running        z3  10.0.81.121  i-09ec0c2e7f3594683  large    true  

1 vms

Succeeded
```

## <div id="3"/> 3. PaaS-TA AP Portal Installation
### <div id="3.1"/> 3.1. Portal App Configuration
9 Portal-related apps are deployed on PaaS-TA AP, and the configuration are as follows.
```
portal-app-1.2.2
├── portal-api-2.4.1
├── portal-common-api-2.2.1
├── portal-gateway-2.1.0
├── portal-log-api-2.1.0
├── portal-registration-2.1.0
├── portal-ssh-1.0.0
├── portal-storage-api-2.2.1
├── portal-web-admin-2.3.1
└── portal-web-user-2.4.1
```
### <div id="3.2"/> 3.2. Portal App Deployment Script Variable Settings
Navigate to the location where the script is located to run the Portal App deployment script.

```
### Change Installation Directory
$ cd ~/workspace/portal-deployment/portal-container-infra/scripts
```

Set the Script variable. (FILE PATH Partial Change Required, Remaining Options)
> $ vi portal-app-variable.yml
```
#!/bin/bash

##FILE PATH
COMMON_VARS_PATH=~/workspace/common/common_vars.yml	# common_vars.yml Path
PORTAL_INFRA_VARS_PATH=~/workspace/portal-deployment/portal-container-infra/vars.yml	# portal_infra_vars.yml Path
PORTAL_APP_WORKING_DIRECTORY=~/workspace/portal-deployment/portal-container-infra/portal-app # Portal APP Working Path


##PORTAL VARIABLE
USER_APP_SIZE_MB=0					# USER My App size(MB), if value==0 -> unlimited
MONITORING_ENABLE=false					# Monitoring Enable Option

PORTAL_ORG_NAME="portal"				# PaaS-TA Portal Org Name
PORTAL_SPACE_NAME="system"				# PaaS-TA Portal Space Name
PORTAL_QUOTA_NAME="portal_quota"			# PaaS-TA Portal Quota Name
PORTAL_SECURITY_GROUP_NAME="portal"			# PaaS-TA Portal Security Group Name

MAIL_SMTP_HOST="smtp.gmail.com"				# Mail-SMTP Host
MAIL_SMTP_PORT="465"					# Mail-SMTP Port
MAIL_SMTP_USERNAME="paasta"				# Mail-SMTP User Name
MAIL_SMTP_PASSWORD="paasta"				# Mail-SMTP Password
MAIL_SMTP_USEREMAIL="paas-ta@gmail.com"			# Mail-SMTP User Email

##PORTAL APP INSTANCES
PORTAL_API_INSTANCE=1					# PORTAL-API INSTANCES
PORTAL_COMMON_API_INSTANCE=1				# PORTAL-COMMON-API INSTANCES
PORTAL_GATEWAY_INSTANCE=1				# PORTAL-GATEWAY INSTANCES
PORTAL_REGISTRATION_INSTANCE=1				# PORTAL-REGISTRATION INSTANCES
PORTAL_STORAGE_API_INSTANCE=1				# PORTAL-STORAGE-API INSTANCES
PORTAL_WEB_ADMIN_INSTANCE=1				# PORTAL-WEB-ADMIN INSTANCES
PORTAL_WEB_USER_INSTANCE=1				# PORTAL-WEB-USER INSTANCES


##UNCHANGE VARIABLE(if defulat install, don't change variable)
PAASTA_CORE_DEPLOYMENT_NAME="paasta"			# PaaS TA AP Deployment Name
PORTAL_INFRA_DEPLOYMENT_NAME="portal-container-infra"	# Portal Container Infra Deployment Name
PAASTA_DATABASE_INSTANCE_NAME="database"		# PaaS TA AP Database Instance Name
PORTAL_INFRA_DATABASE_NAME="infra"			# Portal Container Infra Database Name
UAA_ADMIN_CLIENT_ID="admin"				# UAA Client ID
CC_DB_NAME="cloud_controller"				# PaaS-TA AP CCDB Name
CC_DB_USER_NAME="cloud_controller"			# PaaS-TA AP CCDB ID
UAA_DB_NAME="uaa"					# PaaS-TA AP UAADB Name
UAA_DB_USER_NAME="uaa"					# PaaS-TA AP UAADB ID
UAAC_PORTAL_CLIENT_ID="portalclient"			# UAAC Portal Client ID

IS_PAAS_TA_EXTERNAL_DB=false				# (true or false)
PAAS_TA_EXTERNAL_DB_IP=					# PaaS-TA AP External DB IP
PAAS_TA_EXTERNAL_DB_PORT=				# PaaS-TA AP External DB Port
PAAS_TA_EXTERNAL_DB_KIND=				# PaaS-TA AP External DB Kind(IF USE e.g. postgres or mysql)
IS_PORTAL_EXTERNAL_DB=false				# (true or false)
PORTAL_EXTERNAL_DB_IP=					# Portal External DB IP
PORTAL_EXTERNAL_DB_PORT=				# Portal External DB Port
PORTAL_EXTERNAL_DB_PASSWORD=				# Portal External DB Password
IS_PORTAL_EXTERNAL_STORAGE=false			# (true or false)
PORTAL_EXTERNAL_STORAGE_IP=				# Portal External Storage IP
PORTAL_EXTERNAL_STORAGE_PORT=				# Portal External Storage Port
PORTAL_EXTERNAL_STORAGE_TENANTNAME=			# Portal External Storage Tenant Name
PORTAL_EXTERNAL_STORAGE_USERNAME=			# Portal External Storage Username
PORTAL_EXTERNAL_STORAGE_PASSWORD=			# Portal External Storage Password
```


### <div id="3.3"/> 3.3. Run the Portal App Deployment Script
When the variable setting is complete, run the deployment script.
> $ source deploy-portal-app.sh
```
.....
.....

name                  requested state   processes           routes
portal-api            started           web:1/1, task:0/0   portal-api.61.252.53.246.nip.io
portal-common-api     started           web:1/1, task:0/0   portal-common-api.61.252.53.246.nip.io
portal-gateway        started           web:1/1, task:0/0   portal-gateway.61.252.53.246.nip.io
portal-log-api        started           web:1/1, task:0/0   portal-log-api.61.252.53.246.nip.io
portal-registration   started           web:1/1, task:0/0   portal-registration.61.252.53.246.nip.io
portal-storage-api    started           web:1/1, task:0/0   portal-storage-api.61.252.53.246.nip.io
portal-web-admin      started           web:1/1, task:0/0   portal-web-admin.61.252.53.246.nip.io
portal-web-user       started           web:1/1             portal-web-user.61.252.53.246.nip.io
ssh-app               started           web:1/1             ssh-app.61.252.53.246.nip.io
```



## <div id="4"/>4. PaaS-TA AP Portal Operation

### <div id="4.1"/> 4.1. Enable the user's organizational creation flag

PaaS-TA sets up that ordinary users cannot create an organization. Enable user_org_creation FLAG so that users can create an organization because it is necessary to create an organization and space for portal deployments and to run tests. To activate FLAG, logging in with PaaS-TA Admin(Operator) account is required.

```
$ cf enable-feature-flag user_org_creation
```
```
Setting status of user_org_creation as admin...
OK

Feature user_org_creation Enabled.
```

### <div id="4.2"/> 4.2. Error on user portal UAA page

- Set the endpoint of the uaac and run the uaac login.
```
# endpoint Setting
$ uaac target https://uaa.<DOMAIN> --skip-ssl-validation

# target check
$ uaac target
Target: https://uaa.<DOMAIN>
Context: uaa_admin, from client uaa_admin

# uaac login
$ uaac token client get <UAA_CLIENT_ADMIN_ID> -s <UAA_CLIENT_ADMIN_SECRET>
Successfully fetched token via client credentials grant.
Target: https://uaa.<DOMAIN>
Context: admin, from client admin
```
- redirect error - portalclient not registered 
![paas-ta-portal-31]  
1. If the uaac portal client is not registered, a redirect error occurs as shown on the screen.
2. You must add the portalclient through uaac client add.
> $ uaac client add <PORTAL_UAA_CLIENT_ID> -s <PORTAL_UAA_CLIENT_SECRET> --redirect_uri <PORTAL_WEB_USER_URI>, <PORTAL_WEB_USER_URI>/callback --scope   "cloud_controller_service_permissions.read , openid , cloud_controller.read , cloud_controller.write , cloud_controller.admin" --authorized_grant_types "authorization_code , client_credentials , refresh_token" --authorities="uaa.resource" --autoapprove="openid , cloud_controller_service_permissions.read"  

```
# e.g. Create a portal client account

$ uaac client add portalclient -s clientsecret --redirect_uri "http://portal-web-user.<DOMAIN>, http://portal-web-user.<DOMAIN>/callback" \
--scope "cloud_controller_service_permissions.read , openid , cloud_controller.read , cloud_controller.write , cloud_controller.admin" \
--authorized_grant_types "authorization_code , client_credentials , refresh_token" \
--authorities="uaa.resource" \
--autoapprove="openid , cloud_controller_service_permissions.read"
```

- redirectError - Error registering redirect_uri in the portalclient
![paas-ta-portal-32]  
1. If uri is registered incorrectly in the uaac portal client, a redirect error occurs as shown on that screen.
2. The uri should be modified through the uaac client update.
> $ uaac client update portalclient --redirect_uri "<PORTAL_WEB_USER_URI>, <PORTAL_WEB_USER_URI>/callback"   

```
#  e.g. portal client redirect_uri update
$ uaac client update portalclient --redirect_uri "http://portal-web-user.<DOMAIN>, http://portal-web-user.<DOMAIN>/callback"
```

### <div id="4.3"/> 4.3. Apply Catalog
##### 1. Add Catalog buildpack and servicepack
After installing the Paas-TA Portal, you must register the build pack and service pack on the administrator portal to use it on the user portal.
- [Catalog Image Download](https://nextcloud.paas-ta.org/index.php/s/EmzfJw38H4GQKTr/download)

1. Access the Administrator Portal.(portal-web-admin.\<DOMAIN\>)  
![22](https://user-images.githubusercontent.com/104418463/200229183-e694af80-1d0f-485b-b172-457506058dac.png)
2. Click Operation Management.
![23](https://user-images.githubusercontent.com/104418463/200229202-3166c4ae-5d23-4a9c-a8f8-30fbccd480cd.png)
3. Go to the catalog page.
![24](https://user-images.githubusercontent.com/104418463/200229210-997cad41-5c10-4ab0-95c2-be89158f4993.png)
4. Go to the build pack and service pack detail screen, enter the value in each item column, and click Save.
![25](https://user-images.githubusercontent.com/104418463/200229220-d6e9b7cd-a15d-48df-bd2d-7766c7049a7b.png)
5. Check whether the changed value is applied in the user portal.
![paas-ta-portal-19]   

[paas-ta-portal-01]:./images/Paas-TA-Portal_App_01.png
[paas-ta-portal-15]:./images/Paas-TA-Portal_15.png
[paas-ta-portal-16]:./images/Paas-TA-Portal_16.png
[paas-ta-portal-17]:./images/Paas-TA-Portal_17.png
[paas-ta-portal-18]:./images/Paas-TA-Portal_18.png
[paas-ta-portal-19]:./images/Paas-TA-Portal_19.png
[paas-ta-portal-31]:./images/Paas-TA-Portal_27.jpg
[paas-ta-portal-32]:./images/Paas-TA-Portal_28.jpg

### [Index](https://github.com/PaaS-TA/Guide-eng/blob/master/README.md) > [AP Install](../README.md) > Portal Container Type
