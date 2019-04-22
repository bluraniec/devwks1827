# DEVWKS-1827 - Demystifying NSO
This is a Git repository for the Cisco Live DevNet Workshop 1827 - Demystifying NSO

## Story

Lorem ipsum

## Is there a solution? NSO!

Basics about the architecture: 
- SDN (Northbound/Southbound)
- SERVICE
- NETCONF/RESTCONF/YANG Data Models
- CDB
- FASTMAP
- NEDs

## Lab

Environment: Docker
Core: NSO, Netsim
UI: nodeJS, React

## 1st step - SETUP

2 directories - installation (ncs-4.7.1), running directory (ncs-run) 
source ncsrc
login to NSO
packages reload -> see NEDs
devices check-sync -> get initial config
show devices brief -> see onboarded devices, check if 

definitions: package, NED 

✔ Basic setup is done. NSO works, correct NEDs packages are imported so we can talk to these types of devices and Netsim devices are onboarded.

## 2nd step - SERVICE DESIGN

Table with requirements, parameters.

Commands:

Device Type | IOS  | IOS-XR | Juniper
------------ | ------------- | ------------- | -------------
CONFIG | # access-list | # access-list | # access-list

Parameters:

Device Type | IOS  | IOS-XR | Juniper
------------ | ------------- | ------------- | -------------
NAME | x | x | x
ID | x | x | -

Service skeleton generation with descriptions of the structure and components.  
-> XML TEMPLATE (/template)  
-> YANG DATA MODEL (/src)  
-> ADDITIONAL PYTHON LOGIC (/python)

### A - XML TEMPLATE

Put sample config on NSO and generate XML from it. Parametrize it.

### B - YANG DATA MODEL

Prepare all parameters structure.

### C - PYTHON LOGIC (optional)

If we need to calculate something before append to the template or provide some data to configure from 3rd party.

## 3rd step - SERVICE DEPLOYMENT

Execute on single device -> commit dry-run / commit dry-run outformat native -> commit
Create group and execute on group.

## 4th step - CUSTOM UI

Operator don't want to see the CLI. Operator would love to see a beautiful User Interface.

## SUMMARY

Done! He's happy!

What you've learn:
✔ A  
✔ B  
✔ C  
