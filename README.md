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

Simplify the management of Access Control Lists (ACLs) across network devices. Update ACLs on multiple devices on demand, ensuring that security policies are applied consistently across the entire infrastructure.
NSO has service models corresponding to the desired security policies, and generates ACLs matching the desired policy. Policies are translated into the exact syntax needed for the specific vendors and devices, delivered to the device using the appropriate NED.

Basics about the architecture: 
- SDN (Northbound/Southbound)
- SERVICE
- NETCONF/RESTCONF/YANG Data Models
- CDB
- FASTMAP
- NEDs

## Lab

![Test Topology](/test.png)
**Environment:**  
 Docker Compose - <description>  
**Core:**  
 NSO - <description>  
 Netsim - <description>  
**User Interface:**  
 django - <description>  

## 1st step - SETUP

2 directories:   
- ncs-4.7.1 (installation directory)  
- ncs-run (running directory)  

Login to NSO:
```
ncs_cli -C -u admin
```

Show onboarded devices:
```
admin@ncs# show devices brief
NAME    ADDRESS    DESCRIPTION  NED ID
--------------------------------------------
IOS0    127.0.0.1  -            cisco-ios
IOS1    127.0.0.1  -            cisco-ios
IOSXR0  127.0.0.1  -            cisco-ios-xr
IOSXR1  127.0.0.1  -            cisco-ios-xr
JUN0    127.0.0.1  -            netconf
JUN1    127.0.0.1  -            netconf
```

Show packages (as of now only NED packages):
```
admin@ncs# show packages package package-version
               PACKAGE
NAME           VERSION
------------------------
cisco-ios      6.10
cisco-iosxr    7.7
juniper-junos  4.1
```

Sync initial configuration from devices to NSO:
```
admin@ncs# devices sync-from
sync-result {
    device IOS0
    result true
}
sync-result {
    device IOS1
    result true
}
sync-result {
    device IOSXR0
    result true
}
sync-result {
    device IOSXR1
    result true
}
sync-result {
    device JUN0
    result true
}
sync-result {
    device JUN1
    result true
}
```

Check if configuration is present in NSO:
```
admin@ncs# show running-config devices device IOS0
```

Let's try to change something directly on the device and check if NSO will notice this:
```
root@ad720ce13193:/opt/ncs-run# ncs-netsim cli-i IOS0
admin connected from 127.0.0.1 using console on ad720ce13193
IOS0> en
IOS0# conf t
Enter configuration commands, one per line. End with CNTL/Z.
IOS0(config)# interface GigabitEthernet 0/0
IOS0(config-if)# ip address 1.1.1.1 255.255.255.0
IOS0(config-if)# exit
IOS0(config)# exit
IOS0# exit
```

```
root@ad720ce13193:/opt/ncs-run# ncs_cli -C -u admin
admin@ncs# devices check-sync
sync-result {
    device IOS0
    result out-of-sync
    info got: 8149607fd30f37168047f38f30068de9 expected: 4b2d324443b84e439e3e8c77e9da1687

}
sync-result {
    device IOS1
    result in-sync
}
sync-result {
    device IOSXR0
    result in-sync
}
sync-result {
    device IOSXR1
    result in-sync
}
sync-result {
    device JUN0
    result in-sync
}
sync-result {
    device JUN1
    result in-sync
}
```

```
admin@ncs# devices sync-from
sync-result {
    device IOS0
    result true
}
admin@ncs# show running-config devices device IOS0
````

Important Concepts:  
**package** - basic structure of  
**NED** - Network Element Driver, The NSO device manager is the centre of NSO. The device manager maintains a flat list of all managed devices. NSO keeps the master copy of the configuration for each managed device in CDB. Whenever a configuration change is done to the list of device configuration master copies, the device manager will partition this "network configuration change" into the corresponding changes for the actual managed devices. The device manager passes on the required changes to the NEDs, Network Element Drivers. A NED needs to be installed for every type of device OS, like Cisco IOS NED, Cisco XR NED, Juniper JUNOS NED etc. The NEDs communicate through the native device protocol southbound. 

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

## 3rd step - SERVICE CREATION

Service skeleton generation with descriptions of the structure and components.  
-> XML TEMPLATE (/template)  
-> YANG DATA MODEL (/src)  
-> ADDITIONAL PYTHON LOGIC (/python)

Let's generate skeleton of our service:
```
root@ad720ce13193:/opt/ncs-run# cd packages/
root@ad720ce13193:/opt/ncs-run/packages# ncs-make-package --service-skeleton python-and-template ACL_SERVICE
root@ad720ce13193:/opt/ncs-run/packages# cd ACL_SERVICE
root@ad720ce13193:/opt/ncs-run/packages/ACL_SERVICE# ls
README  package-meta-data.xml  python  src  templates  test
```
Generated structure looks following:
```
/ncs-run/packages
├── ACL_SERVICE
│   ├── templates
│   │   └── ACL_SERVICE-template.xml
│   ├── src
│   │   ├── yang
│   │   │   └── ACL_SERVICE.yang
│   │   └── Makefile
│   ├── python
│   │   └── ACL_SERVICE
│   │   │   ├── main.py
│   │   │   └── __init__.py
│   ├── test
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
        └── Makefile
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
            ├── main.py
            └── __init__.py
```


## 4th step - SERVICE DEPLOYMENT

Execute on single device -> commit dry-run / commit dry-run outformat native -> commit
Create group and execute on group.

## 5th step - CUSTOM UI

Operator doesn't want to see the CLI. Operator would love to see a beautiful User Interface.
You can create many various applications on top of NSO.
```
/nso-ui
├── Dockerfile
├── requirements.txt
├── manage.py
├── portal
    ├── settings.py
    ├── urls.py
    ├── wsgi.py
    └── __init__.py
└── app
    ├── admin.py
    ├── apps.py
    ├── models.py
    ├── tests.py
    ├── views.py
    ├── migrations
    └── templates
        └── index.html
       
```


## Summary

:clap: :tada: Done! He's happy!

What you've learnt:  
:white_check_mark: A  
:white_check_mark: B  
:white_check_mark: C. 
