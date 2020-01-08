## Read the Docs

The northbound api doc html/pdf is in the NSO installation directory in the `docs` directory (which includes both NETCONF, JSON-RPC and REST/RESTCONF examples), and the examples.ncs has an example as well `examples.ncs/getting-started/developing-with-ncs/13-rest` or `examples.ncs/getting-started/developing-with-ncs/13-restconf`.

FYI if you are not using vagrant, make sure your NSO application configuration file has REST enabled:
```xml
                <restconf>
                     <enabled>true</enabled>
                </restconf>
                   <webui>
                     <enabled>false</enabled>
                     <transport>
                       <tcp>
                         <enabled>true</enabled>
                         <ip>0.0.0.0</ip>
                         <port>8080</port>
                       </tcp>
                     </transport>
                    </webui>
```

### How to run show commands (on paid / full version of the NED)
[example from developer hub](https://community.cisco.com/t5/nso-developer-hub-discussions/run-show-commands-from-nso-restconf/td-p/3775888):
```
curl -i -u admin:admin  http://localhost:8080/restconf/operations/devices/device=xr-0/live-status/tailf-ned-cisco-ios-xr-stats:exec/any -X POST -H 'Accept: application/yang-data+json' -H 'Content-Type: application/yang-data+json' -T show-run-vrf.json

cat show-run-vrf.json
```
```json
{
  "input":
  {
    "args": "show run vrf foo"
  }
}
```

### Tips from the Doc Guide

The RESTCONF URI base path is:
`http://IP:PORT/restconf` to hit the rest api at 8080 for default config. 

#### FYI NSO RESTCONF Headers
application/yang-data+xml
application/yang-data+json

## filter to non config

In RESTCONF there is only one datastore, the Unified Datastore, which hides the details of a possible candidate/startup datastore. Per default, it returns both configuration and non-configuration data; something that can be changed by using the content query parameter.

/restconf/data?content=nonconfig


### Yang list

When a list key (or keys) are included in the URI, there is a difference in how they are represented.
Assume we have the following list defined in our Yang model:

```yang
                   list mylist {
                     key "k1 k2";
                     leaf k1 {type string;}
                     leaf k2 {type string;}
                     leaf stuff {type string;}
}
```
To address the list entry where k1="one" and k2="two" 
/api/running/mylist=one,two

### operations
Yang RPC operations are found under /restconf/operations/ .

## restconf query options

`depth`
With the RESTCONF depth query parameter you the server. To get the exact same behavior as deep in REST, you need to set: depth=unbounded (which actually is the default). To get the exact same behavior as shallow in REST, you need to set: depth=1 .

`dry-run`
`dry-run-reverse`

## List keys with / in them, like an interface with "0/0"

see this [post](https://community.cisco.com/t5/nso-developer-hub-blogs/nso-northbound-api-restconf-with-yang-list-with-backslash/ba-p/3881754#M234)

### leaf list get

If we, for example, make a GET request to retrieve some leaf-list values, the result (a number of leaf-list nodes) can not be represented in well formed XML (which require a root node of the XML document).

The RESTCONF RFC 8040 (chapter 4.3) explicitly states that: More than one element MUST NOT be
returned for XML encoding.
However, the Cisco RESTCONF API makes use of the same media type as REST to get around this.
Assume we have the following leaf-list defined in our Yang model:
leaf-list myleaflist {type string;}
We can now retrieve the leaf-list values in XML format as a collection:
GET /restconf/data/myleaflist HTTP/1.1
Accept: application/vnd.yang.collection+xml
The returned reply may look something like:
```xml
<collection xmlns="http://tail-f.com/ns/restconf/collection/1.0">
  <myleaflist xmlns="urn:foo">one</myleaflist>
  <myleaflist xmlns="urn:foo">two</myleaflist>
  <myleaflist xmlns="urn:foo">three</myleaflist>
```

### Query API
Query API where you can retrieve data based on an XPath expression and other selection filters.
'http://admin:admin@localhost:8080/restconf/tailf/query' \
                        -X POST -T test.xml \
                        -H "Content-Type: application/yang-data+xml"
