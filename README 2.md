# NSO Northbound REST / RESTCONF Examples

The parlance of network engineers expecting a platform having an API is to look at the REST API first, and  having a simple requests demo helps folks getting started. 

## REST API vs RESTCONF API
First off, the NSO BU recommends using the RESTCONF API instead of the REST API. The NSO application has both, the REST API was created before the RESTCONF industry RFC standards were created. After the RESTCONF standards were set, the focus and recommendation for using the NSO REST based API is to use the RESTCONF, not the REST API. They are different northbound APIs, and both still exist (at the time of this writing), but for all intensive purposes, you should be starting your development against the RESTCONF one, not the REST one. 

`From the Northbound API Guide`: **TLDR: The REST API is deprecated and will be removed in NSO 5.3, use the RESTCONF API instead.** [REST vs RESTCONF on NSO](https://community.cisco.com/t5/nso-developer-hub-discussions/difference-between-rest-and-restconf/td-p/3506582)

## My Demo Setup

I am using the [NSO Vagrant](https://github.com/NSO-developer/nso-vagrant) local install NSO, with NSO version `5.1.0.1` on Ubuntu, from my Macbook Pro  OS X `10.14.5`. The Vagrant File sets up a few netsims with the non-production dummy NEDs (`cisco-ios-cli-3.8` rather than `cisco-ios`) which come for free with the NSO install. I will use these NEDs for the example, but remember they should not be used on real devices. 

I am using Postman Version 6.7.4 on my laptop on the OS X side, running it against the port exposed from the virtual machine. You will need to import the collection, and may need to change the `NSO IP` and `Port`. The default port for NSO is `8080`, but since I am using vagrant it is saved as `8009` in the collection. 


## Importing the Postman Collection

I created a Postman collection, which is an organized set of RESTCONF API calls to your NSO server. You can learn how to import the collection [here](https://learning.getpostman.com/docs/postman/collections/data_formats/#importing-postman-data). The collection is in this repository called `NSO RESTCONF NORTHBOUND API.postman_collection.json`. 

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

### Setting The Stage

First I added the netsim devices, which were created from the vagrant set up to the running NSO CDB:
```bash
nso-vagrant$ vagrant ssh
Welcome to Ubuntu 18.04.2 LTS (GNU/Linux 4.15.0-51-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Fri Jun 28 17:57:13 UTC 2019

  System load:  0.09              Processes:           98
  Usage of /:   6.3% of 61.80GB   Users logged in:     0
  Memory usage: 26%               IP address for eth0: 10.0.2.15
  Swap usage:   0%                IP address for eth1: 192.168.56.200


32 packages can be updated.
20 updates are security updates.


Last login: Fri Jun 28 17:53:33 2019 from 10.0.2.2
virtualenvwrapper.user_scripts creating /home/vagrant/.virtualenvs/premkproject
virtualenvwrapper.user_scripts creating /home/vagrant/.virtualenvs/postmkproject
virtualenvwrapper.user_scripts creating /home/vagrant/.virtualenvs/initialize
virtualenvwrahistpper.user_scripts creating /home/vagrant/.virtualenvs/premkvirtualenv
virtualenvwrapper.user_scripts creating /home/vagrant/.virtualenvs/postmkvirtualenv
virtualenvwrapper.user_scripts creating /home/vagrant/.virtualenvs/prermvirtualenv
virtualenvwrapper.user_scripts creating /home/vagrant/.virtualenvs/postrmvirtualenv
virtualenvwrapper.user_scripts creating /home/vagrant/.virtualenvs/predeactivate
virtualenvwrapper.user_scripts creating /home/vagrant/.virtualenvs/postdeactivate
virtualenvwrapper.user_scripts creating /home/vagrant/.virtualenvs/preactivate
virtualenvwrapper.user_scripts creating /home/vagrant/.virtualenvs/postactivate
virtualenvwrapper.user_scripts creating /home/vagrant/.virtualenvs/get_env_details
vagrant@vagrant:~$
vagrant@vagrant:~$
vagrant@vagrant:~$ ls
nso-install  nso-run
vagrant@vagrant:~$ kickoffnso

admin connected from 10.0.2.2 using ssh on vagrant
admin@ncs# conf
Entering configuration mode terminal
admin@ncs(config)# devices device
admin@ncs# exit
vagrant@vagrant:~$ cd nso-run
vagrant@vagrant:nso-run$ ncs-netsim list
ncs-netsim list for  /home/vagrant/nso-run/netsim

name=netsim-ios-0 netconf=12022 snmp=11022 ipc=5010 cli=10022 dir=/home/vagrant/nso-run/netsim/netsim-ios-/netsim-ios-0
name=netsim-ios-1 netconf=12023 snmp=11023 ipc=5011 cli=10023 dir=/home/vagrant/nso-run/netsim/netsim-ios-/netsim-ios-1
name=netsim-ios-2 netconf=12024 snmp=11024 ipc=5012 cli=10024 dir=/home/vagrant/nso-run/netsim/netsim-ios-/netsim-ios-2
vagrant@vagrant:nso-run$
admin@ncs(config)# devices device netsim-ios-0 address 127.0.0.1 port 10022 device-type cli ned-id cisco-ios?
Possible completions:
  cisco-ios-cli-3.8  cisco-iosxr-cli-3.5
admin@ncs(config)# devices device netsim-ios-0 address 127.0.0.1 port 10022 device-type cli ned-id cisco-ios-cli-3.8
admin@ncs(config-device-netsim-ios-0)# authgroup default state admin-state unlocked
admin@ncs(config-device-netsim-ios-0)# commit
Commit complete.
admin@ncs(config-device-netsim-ios-0)# devices device netsim-ios-1 address 127.0.0.1 port 10023 device-type cli ned-id cisco-ios-cli-3.8
admin@ncs(config-device-netsim-ios-1)# authgroup default state admin-state unlocked  
admin@ncs(config-device-netsim-ios-1)# devices device netsim-ios-2 address 127.0.0.1 port 10024 device-type cli ned-id cisco-ios-cli-3.8                       
admin@ncs(config-device-netsim-ios-2)# authgroup default state admin-state unlocked                         
admin@ncs(config-device-netsim-ios-2)# commit
Commit complete.
admin@ncs(config-device-netsim-ios-2)# end
admin@ncs# devices device netsim-ios-1 ssh fetch-host-keys
result failed
info Failed to connect to device netsim-ios-1: connection refused
admin@ncs# exit
vagrant@vagrant:nso-run$ ncs-netsim status
DEVICE netsim-ios-0
connection refused (status)
DEVICE netsim-ios-1
connection refused (status)
DEVICE netsim-ios-2
connection refused (status)
vagrant@vagrant:nso-run$ ncs-netsim start
DEVICE netsim-ios-0 OK STARTED
DEVICE netsim-ios-1 OK STARTED
DEVICE netsim-ios-2 OK STARTED
vagrant@vagrant:nso-run$ kickoffnso

admin connected from 10.0.2.2 using ssh on vagrant
admin@ncs# devices device netsim-ios-0 ssh fetch-host-keys
result updated
fingerprint {
    algorithm ssh-rsa
    value c6:bb:cc:b0:3a:25:44:30:59:58:c8:46:08:d9:83:4e
}
admin@ncs# devices device netsim-ios-1 ssh fetch-host-keys
result updated
fingerprint {
    algorithm ssh-rsa
    value c6:bb:cc:b0:3a:25:44:30:59:58:c8:46:08:d9:83:4e
}
admin@ncs# devices device netsim-ios-2 ssh fetch-host-keys
result updated
fingerprint {
    algorithm ssh-rsa
    value c6:bb:cc:b0:3a:25:44:30:59:58:c8:46:08:d9:83:4e
}
admin@ncs# devices device netsim-ios-0 ssh fetch-host-keys
admin@ncs# devices sync-from
sync-result {
    device netsim-ios-0
    result true
}
sync-result {
    device netsim-ios-1
    result true
}
sync-result {
    device netsim-ios-2
    result true
}
admin@ncs#
```

## How I built the example XML payloads 

### Adding a Device 

**Get the XML for the payload from commit dry-run outformat xml**
```
admin@ncs(config)# devices device netsim-ios-0 authgroup default address 127.0.0.1 port 10022 device-type cli ned-id cisco-ios-cli-3.8
admin@ncs(config-device-netsim-ios-0)# state admin-state unlocked
admin@ncs(config-device-netsim-ios-0)# commit dry-run outformat xml
result-xml {
    local-node {
        data <devices xmlns="http://tail-f.com/ns/ncs">
               <device>
                 <name>netsim-ios-0</name>
                 <address>127.0.0.1</address>
                 <port>10022</port>
                 <authgroup>default</authgroup>
                 <device-type>
                   <cli>
                     <ned-id xmlns:cisco-ios-cli-3.8="http://tail-f.com/ns/ned-id/cisco-ios-cli-3.8">cisco-ios-cli-3.8:cisco-ios-cli-3.8</ned-id>
                   </cli>
                 </device-type>
                 <state>
                   <admin-state>unlocked</admin-state>
                 </state>
               </device>
             </devices>
    }
}
admin@ncs(config-device-netsim-ios-0)#
```

**payload is** 

```xml

               <device >
                 <name>netsim-ios-0</name>
                 <address>127.0.0.1</address>
                 <port>10022</port>
                 <authgroup>default</authgroup>
                 <device-type>
                   <cli>
                     <ned-id xmlns:cisco-ios-cli-3.8="http://tail-f.com/ns/ned-id/cisco-ios-cli-3.8">cisco-ios-cli-3.8:cisco-ios-cli-3.8</ned-id>
                   </cli>
                 </device-type>
                 <state>
                   <admin-state>unlocked</admin-state>
                 </state>
               </device>
```
**at this path**
`http://{{NSO_IP}}:{{NSO_HTTP_PORT}}/restconf/data/tailf-ncs:devices` with POST method

### Edit something on the Device config

**Get the XML from the CLI**

```
admin@ncs(config)# devices device netsim-ios-0 config
admin@ncs(config-if)# ip address 10.0.0.5 255.255.255.255
admin@ncs(config-if)# commit dry-run
admin@ncs(config-if)# commit dry-run outformat xml
result-xml {
    local-node {
        data <devices xmlns="http://tail-f.com/ns/ncs">
               <device>
                 <name>netsim-ios-0</name>
                 <config>
                   <interface xmlns="urn:ios">
                     <Ethernet>
                       <name>0/0</name>
                       <ip>
                         <address>
                           <primary>
                             <address>10.0.0.5</address>
                             <mask>255.255.255.255</mask>
                           </primary>
                         </address>
                       </ip>
                     </Ethernet>
                   </interface>
                 </config>
               </device>
             </devices>
    }
}
admin@ncs(config-if)#
```

**Put into the payload at the right path**


