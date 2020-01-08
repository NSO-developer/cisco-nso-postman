# Postman Collection for Cisco NSO

This collection is used as the source to auto-generate the [swagger example API examples](https://developer.cisco.com/docs/nso/#!cisco-nso-swagger-api-docs). 

It demonstrates basic interactions that commonly occur with NSO. It is not comprehensive, but meant as a learning tool for beginners. 

> Note: Depending on what version of NSO you are using and what version of NEDs you are using, you may have slightly different paths and payloads. This is meant to reflect what is the current freely available version of NSO and NEDs. 

> Also Note: Currently this collection only shows IOS device management, but you could easily extend it to the other NEDs as well (ASA, Nexus, IOS-XR, etc). 

## Installation

I am using the [NSO Vagrant](https://github.com/NSO-developer/nso-vagrant) local install NSO, with NSO version `5.3` on Ubuntu, from my Macbook Pro  OS X `10.14.5`. I am using the following NEDs (which are [freely downloadable from the NSO page where you download the installer](https://developer.cisco.com/docs/nso/#!getting-nso)): `cisco-asa-cli-6.7  cisco-ios-cli-6.42  cisco-iosxr-cli-7.18  cisco-nx-cli-5.13`

I am using Postman Version 6.7.4 on my laptop on the OS X side, running it against the port exposed from the virtual machine. You will need to import the collection, and may need to change the `NSO IP` and `Port`. The default port for NSO is `8080`, but since I am using vagrant it is saved as `8009` in the collection. 

### Importing the Postman Collection

I created a Postman collection, which is an organized set of RESTCONF API calls to your NSO server. You can learn how to import the collection [here](https://learning.getpostman.com/docs/postman/collections/data_formats/#importing-postman-data). The collection is in this repository called `NSO Sample API Requests.postman_collection.json`. 

Once you import the collection, you may want to change the collection variables I made for the `NSO IP` and `Port`. I used localhost and the vagrant port for my set up, if you are not running on vagrant you should likely update the port to `8080`. [See instructions here for changing collection variables](https://learning.getpostman.com/docs/postman/environments_and_globals/variables#defining-collection-variables).

### Ports and Vagrant
If you are using vagrant, check what ports are mapped for the HTTP API (usually 8080) so your REST call path can be correct.
For example:
```
admin@ncs# exit
vagrant@vagrant:nso-run$ exit
logout
Connection to 127.0.0.1 closed.
nso-vagrant$ vagrant port
The forwarded ports for the machine are listed below. Please note that
these values may differ from values configured in the Vagrantfile if the
provider supports automatic port collision detection and resolution.

    22 (guest) => 2222 (host)
  4000 (guest) => 4009 (host)
  5000 (guest) => 5003 (host)
  8080 (guest) => 8009 (host)
    80 (guest) => 8089 (host)
   443 (guest) => 8449 (host)
nso-vagrant$
```

### Adding Dummy Netsim Devices and Device-Groups

First I created the netsim devices in the vagrant bash shell, then get into the NSO CLI and add them to the application. Adjust your syntax based on the package version you are using for the neds. 

```
ncs-netsim create-network cisco-ios-cli-6.42 1 ios
ncs-netsim add-to-network cisco-nx-cli-5.13 1 nx
ncs-netsim add-to-network cisco-iosxr-cli-7.18 1 iosxr
ncs-netsim add-to-network cisco-asa-cli-6.7 1 asa
ncs-netsim start
ncs-netsim list

ncs_cli -C -u admin
config


devices device ios0 authgroup default address 127.0.0.1 port 10022 device-type cli ned-id cisco-ios-cli-6.42
state admin-state unlocked
exit
devices device nx0 authgroup default address 127.0.0.1 port 10023 device-type cli ned-id cisco-nx-cli-5.13
state admin-state unlocked
exit
devices device iosxr0 authgroup default address 127.0.0.1 port 10024 device-type cli ned-id cisco-iosxr-cli-7.18
state admin-state unlocked
exit
devices device asa0 authgroup default address 127.0.0.1 port 10025 device-type cli ned-id cisco-asa-cli-6.7
state admin-state unlocked
commit dry-run outformat xml
commit
top
devices fetch-ssh-host-keys
devices sync-from
devices device-group campus-devices device-name [ ios0 nx0 ]
top
devices device-group backbone-devices device-name [ iosxr0 asa0 ]
top
devices device-group parent-group-all-devices device-group [ campus-devices backbone-devices ]
commit 
end
exit
```

For the dummy service example, I made only a single change to it after using the `ncs-make-package` bash command in the `nso-run/packages` directory. I added the following XML template:
```xml
cd $HOME/nso-run/packages
ncs-make-package --service-skeleton template simple-sample-snmp-server-service

vim simple-sample-snmp-server-service/templates/simple-sample-snmp-server-service-template.xml

<config-template xmlns="http://tail-f.com/ns/config/1.0"
                 servicepoint="simple-sample-snmp-server-service">
  <devices xmlns="http://tail-f.com/ns/ncs">
    <device>
      <!--
          Select the devices from some data structure in the service
          model. In this skeleton the devices are specified in a leaf-list.
          Select all devices in that leaf-list:
      -->
      <name>{/device}</name>
      <config>
                         <snmp-server xmlns="urn:ios">
                     <host>
			     <ip-address>{/dummy}</ip-address>
                       <message-type>traps</message-type>
                       <community-string>NSO-COMM-STRING</community-string>
                     </host>
                   </snmp-server>

      </config>
    </device>
  </devices>
</config-template>
vagrant@vagrant:packages$

```


## Author(s)

This project was written and is maintained by the following individuals:

* Jason Belk <jabelk@cisco.com>
