# Postman Collection for Cisco NSO

This collection is used as the source to auto-generate the [swagger example API examples](https://developer.cisco.com/docs/nso/#!cisco-nso-swagger-api-docs). 

It demonstrates basic interactions that commonly occur with NSO. It is not comprehensive, but meant as a learning tool for beginners. 

> Note: Depending on what version of NSO you are using and what version of NEDs you are using, you may have slightly different paths and payloads. This is meant to reflect what is the current freely available version of NSO and NEDs. 

> Also Note: Currently this collection only shows IOS device management, but you could easily extend it to the other NEDs as well (ASA, Nexus, IOS-XR, etc). 

## Installation

I am using the [NSO Reservable Sandbox](https://blogs.cisco.com/developer/nso-learning-lab-and-sandbox) system install NSO, with NSO version `5.3.0.1`. 

I have included two [Postman environments](https://learning.postman.com/docs/sending-requests/managing-environments/), one for the already configured Prod System Install, and one assuming you set up the local install yourself. The least effort to learn the API is using the system install, as NSO is already set up for you. 

### Importing the Postman Collection

I created a Postman collection, which is an organized set of RESTCONF API calls to your NSO server. You can learn how to import the collection [here](https://learning.getpostman.com/docs/postman/collections/data_formats/#importing-postman-data). The collection is in this repository called `NSO Sample API Requests.postman_collection.json`. 


For the dummy service example, I made two small changes to it after using the `ncs-make-package` bash command in the `nso-run/packages` directory. I added the following XML template:
```bash
cd /var/opt/ncs/packages
ncs-make-package --service-skeleton template snmp-servers
rm snmp-servers/src/yang/snmp-servers.yang
vi snmp-servers/src/yang/snmp-servers.yang

** copy and paste my file below in **
https://github.com/jabelk/snmp-servers/blob/main/src/yang/snmp-servers.yang#L31

cd snmp-servers/src/
make
cd ..
cd templates/
rm snmp-servers-template.xml
vi snmp-servers-template.xml

** add in the template (copy and paste file below), remove old file **

https://github.com/jabelk/snmp-servers/blob/main/templates/snmp-servers-template.xml#L19

log into NSO
ncs_cli -C
packages reload

```

## Author(s)

This project was written and is maintained by the following individuals:

* Jason Belk <jabelk@cisco.com>
