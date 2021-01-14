# Postman Collection for Cisco NSO

This collection is designed to be used with the NSO DevNet Sandboxes:

- Always-ON Sandbox (READ ONLY)
- [Reservable Sandbox Prod (System Install)](https://devnetsandbox.cisco.com/RM/Diagram/Index/43964e62-a13c-4929-bde7-a2f68ad6b27c?diagramType=Topology)
- Reservable Sandbox Dev (Local Install)

This collection is used as the source to auto-generate the [swagger example API examples](https://developer.cisco.com/docs/nso/#!cisco-nso-swagger-api-docs). 

It demonstrates basic interactions that commonly occur with NSO. It is not comprehensive, but meant as a learning tool for beginners. 

> Note: Depending on what version of NSO you are using and what version of NEDs you are using, you may have slightly different paths and payloads. This is meant to reflect what is the current freely available version of NSO and NEDs. 

> Also Note: Currently this collection only shows IOS device management, but you could easily extend it to the other NEDs as well (ASA, Nexus, IOS-XR, etc). 

## Installation

I am using the [NSO Reservable Sandbox](https://blogs.cisco.com/developer/nso-learning-lab-and-sandbox) system install NSO, with NSO version `5.4.1`. 

I have included **three** [Postman environments](https://learning.postman.com/docs/sending-requests/managing-environments/), for each of the NSO Sandbox environments. 

I created a Postman collection, which is an organized set of RESTCONF API calls to your NSO server. You can learn how to import the collection [here](https://learning.getpostman.com/docs/postman/collections/data_formats/#importing-postman-data). The collection is in this repository called `NSO Sample API Requests.postman_collection.json`. 



## Author(s)

This project was written and is maintained by the following individuals:

* Jason Belk <jabelk@cisco.com>
