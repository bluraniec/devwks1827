# DEVWKS-1827 - Demystifying NSO
This is a Git repository for the Cisco Live DevNet Workshop 1827 - Demystifying NSO

## Story
> Carl is a Network Engineer in the company Live! He has received an urgent task to deploy an ACL with several statements on 347 devices.

Difficulties:  
- urgent task = time constraint
- 3 different types of devices: IOS, IOS-XR and Juniper, he is not extremely fluent in each
- tons of devices to put the config into it
- the ACL is going to be deployed there only for 2 weeks, for the purpose of MWs, some changes in the network… and then it has to be again removed from all devices
- Carl is going at that time for his PTO and there is no one who has enough networking knowledge to remove this configuration from all different vendors devices

IT’S A NIGHTMARE!! But it might happen...

## Is there a solution? NSO!

In this lab we're going to help Carl with his task using NSO - Network Services Orchestrator.  

Basics about the architecture: 
- SDN (Northbound/Southbound)
- SERVICE
- NETCONF/RESTCONF/YANG Data Models
- CDB
- FASTMAP
- NEDs

## Lab

**Environment:** Docker Compose  
**Core:** NSO, Netsim  
**User Interface:** nodeJS, React  

## 1st step - SETUP

2 directories:   
- ncs-4.7.1 (installation directory)  
- ncs-run (running directory)  

Source:
```
source ncsrc
```

Login to NSO:
```
ncs_cli -C -u admin
```

Show onboarded devices:
```
show devices brief
```
packages reload -> see NEDs
devices check-sync -> get initial config

**package** - basic structure of  
**NED** - Network Element Driver

✔ Basic setup is done. NSO works, correct NEDs packages are imported so we can talk to these types of devices and Netsim devices are onboarded.

## 2nd step - SERVICE DESIGN

Table with requirements, parameters.

Commands:

Device Type | IOS  | IOS-XR | Juniper
------------ | ------------- | ------------- | -------------
CONFIG | # access-list | # ipv4 access-list CLUS_BLOCK  
x | # access-list |   10 permit 172.16.0.0 0.0.255.255  
x | # access-list |   20 deny 192.168.34.0 0.0.0.255  
x | # access-list |   30 permit  | # access-list

Parameters:

Device Type | IOS  | IOS-XR | Juniper
------------ | ------------- | ------------- | -------------
NAME | x | x | x
ID | x | x | -

Service skeleton generation with descriptions of the structure and components.  
-> XML TEMPLATE (/template)  
-> YANG DATA MODEL (/src)  
-> ADDITIONAL PYTHON LOGIC (/python)

```
/ncs-run/packages
├── ACL_SERVICE
│   ├── templates
│   │   └── ACL_SERVICE-template.xml
│   ├── src
│   │   ├── yang
│   │   │   └── ACL_SERVICE.yang
│   │   ├── Makefile
│   │   └── java
│   ├── python
│   │   └── ACL_SERVICE
│   │   │   ├── ACL_SERVICE.py
│   │   │   └── __init__.py
│   ├── test
│   ├── load-dir
│   ├── package-meta-data.xml
│   └── README
└── OTHER_SERVICE
```

### A - XML TEMPLATE

First part is an XML template of our Service - this will be an actual config pushed to the device.
```
/ncs-run/packages
├── ACL_SERVICE
    └── templates
        └── ACL_SERVICE-template.xml
```
Put sample config on NSO and generate XML from it. Parametrize it.

### B - YANG DATA MODEL

Second part is a YANG Data Model of our Service - this will give us an opportunity to provide the CLI interface to configure our Service. Prepare all parameters structure.
```
/ncs-run/packages
├── ACL_SERVICE
    └── src
        ├── yang
        │   └── ACL_SERVICE.yang
        ├── Makefile
        └── java
```

After creation of the model, go back to the ACL_SERVICE folder and compile the package executing command:
```
make clean all
```

### C - PYTHON LOGIC (optional)

Third part is a Python code of our Service - this will give us an opportunity to provide logic to our Service if we need to calculate something before appending to the template or provide some data to configure from 3rd party.
```
/ncs-run/packages
├── ACL_SERVICE
    └── python
        └── ACL_SERVICE
            ├── ACL_SERVICE.py
            └── __init__.py
```


## 3rd step - SERVICE DEPLOYMENT

Execute on single device -> commit dry-run / commit dry-run outformat native -> commit
Create group and execute on group.

## 4th step - CUSTOM UI

Operator don't want to see the CLI. Operator would love to see a beautiful User Interface.

## Summary

:clap: :tada: Done! He's happy!

What you've learn:  
:white_check_mark: A  
:white_check_mark: B  
:white_check_mark: C. 
