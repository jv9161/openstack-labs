## NEUTRON on a freshly bootstrapped cloud 
So you have created your first OpenStack clound and you want to build networks. The first step is to build the core network, which most people name the "provider" network. (Would the name **"fabric"** be better here???) 

This ***MUST*** be done with *admin* rights using the commmand line because Stu configured the `/etc/neutron/plugins/ml2ml2_conf.ini` file to restrict normal users to VXLAN only, while the admin can configure both FLAT and VXLAN. For our provider network, which we are calling *fabric*, we need a flat network, so here is the command to create the provider network:

 `neutron net-create --shared --provider:physical_network provider --provider:network_type flat provider`  
```
Created a new network:
+---------------------------+--------------------------------------+
| Field                     | Value                                |
+---------------------------+--------------------------------------+
| admin_state_up            | True                                 |
| availability_zone_hints   |                                      |
| availability_zones        |                                      |
| created_at                | 2016-08-30T19:43:59                  |
| description               |                                      |
| id                        | aed6e80e-f6c3-42c2-ad89-0d06c188d30b |
| ipv4_address_scope        |                                      |
| ipv6_address_scope        |                                      |
| mtu                       | 1500                                 |
| name                      | provider                             |
| port_security_enabled     | True                                 |
| provider:network_type     | flat                                 |
| provider:physical_network | provider                             |
| provider:segmentation_id  |                                      |
| router:external           | False                                |
| shared                    | True                                 |
| status                    | ACTIVE                               |
| subnets                   |                                      |
| tags                      |                                      |
| tenant_id                 | 810f65ae070847a2851d69344b1fad25     |
| updated_at                | 2016-08-30T19:43:59                  |
+---------------------------+--------------------------------------+
```
**Fast translation:**  
 `create new network, access=everyone, NIC=eth0, layer2=no vlan tags, named"provider"`  
 
**Rocket Scientist command breakdown:**  
  **`neutron net-create`** Create a virtual network  
  **`--shared`** Access = everyone (all tenants)  
  **`--provider:physical_network`**  *`provider`*  Means the provider network's physical interface is a NIC called  *`"provider"`* which in our case maps to physical `eth0` of the node.  Here is how you determine that: 
1. The name of the NIC is arbitrarily named *`"provider"`*
2. Now inspect `/etc/neutron/plugins/ml2/linuxbridge_agent.ini` file (assuming you are using linuxbridge): 
3. Look for: **physical_interface_mappings** = **provider**:**eth0** (See example below) 
4. Therefore, the interface called *`"provider"`* = **eth0** in this example
5. **Therefore** `--provider:physical_network provider` translates in this example to **"use eth0"**

 Here is an excerpt you would need to find in *`/etc/neutron/plugins/ml2/linuxbridge_agent.ini`* to figure this out.  
   ```
    [linux_bridge]  
    physical_interface_mappings = provider:eth0
   ```
   
   **`--provider:network_type flat`** - Options are: FLAT (no vlan tags), VLAN, VXLAN, and GRE.   
   **`provider`** - The name of this new virtual network

### Create the provider subnet
   `neutron subnet-create --name provider --allocation-pool start=10.3.200.100,end=10.3.203.250 --dns-nameserver 10.3.200.1 --gateway 10.3.200.1  provider 10.3.200.100/22`

```
Created a new subnet:
+-------------------+--------------------------------------------------+
| Field             | Value                                            |
+-------------------+--------------------------------------------------+
| allocation_pools  | {"start": "10.3.200.100", "end": "10.3.203.250"} |
| cidr              | 10.3.200.0/22                                    |
| created_at        | 2016-08-30T19:52:22                              |
| description       |                                                  |
| dns_nameservers   | 10.3.200.1                                       |
| enable_dhcp       | True                                             |
| gateway_ip        | 10.3.200.1                                       |
| host_routes       |                                                  |
| id                | 25e05135-9d21-436c-8934-a670da127119             |
| ip_version        | 4                                                |
| ipv6_address_mode |                                                  |
| ipv6_ra_mode      |                                                  |
| name              | provider                                         |
| network_id        | aed6e80e-f6c3-42c2-ad89-0d06c188d30b             |
| subnetpool_id     |                                                  |
| tenant_id         | 810f65ae070847a2851d69344b1fad25                 |
| updated_at        | 2016-08-30T19:52:22                              |
+-------------------+--------------------------------------------------+
```

`source alice.rc`

### Create a student net
`neutron net-create student01`

   ```
 Created a new network:
+---------------------------+--------------------------------------+
| Field                     | Value                                |
+---------------------------+--------------------------------------+
| admin_state_up            | True                                 |
| availability_zone_hints   |                                      |
| availability_zones        |                                      |
| created_at                | 2016-08-31T00:13:12                  |
| description               |                                      |
| id                        | 87361f8f-cad4-477e-81d1-7b9b8e2cc4d9 |
| ipv4_address_scope        |                                      |
| ipv6_address_scope        |                                      |
| mtu                       | 1450                                 |
| name                      | student01                            |
| port_security_enabled     | True                                 |
| provider:network_type     | vxlan                                |
| provider:physical_network |                                      |
| provider:segmentation_id  | 83                                   |
| router:external           | False                                |
| shared                    | False                                |
| status                    | ACTIVE                               |
| subnets                   |                                      |
| tags                      |                                      |
| tenant_id                 | 810f65ae070847a2851d69344b1fad25     |
| updated_at                | 2016-08-31T00:13:13                  |
+---------------------------+--------------------------------------+
  ```
### Create  student subnet
`neutron subnet-create --name student01 --dns-nameserver 8.8.8.8 --gateway 172.16.1.1 student01 172.16.1.0/24`  
 `--name NAME`           Name of this subnet  
 `--dns-nameserver DNS_NAMESERVER` DNS name server for this subnet (This option can be repeated).  
 `--gateway GATEWAY_IP`  Gateway IP of this subnet.  
 `NETWORK` Network ID or name this subnet belongs to.  
 `CIDR`  CIDR of subnet to create.

   ```
Created a new subnet:
+-------------------+------------------------------------------------+
| Field             | Value                                          |
+-------------------+------------------------------------------------+
| allocation_pools  | {"start": "172.16.1.2", "end": "172.16.1.254"} |
| cidr              | 172.16.1.0/24                                  |
| created_at        | 2016-08-31T00:14:36                            |
| description       |                                                |
| dns_nameservers   | 8.8.8.8                                        |
| enable_dhcp       | True                                           |
| gateway_ip        | 172.16.1.1                                     |
| host_routes       |                                                |
| id                | 998ec3dc-6497-46d8-ad7e-7e7dd73340b6           |
| ip_version        | 4                                              |
| ipv6_address_mode |                                                |
| ipv6_ra_mode      |                                                |
| name              | student01                                      |
| network_id        | 87361f8f-cad4-477e-81d1-7b9b8e2cc4d9           |
| subnetpool_id     |                                                |
| tenant_id         | 810f65ae070847a2851d69344b1fad25               |
| updated_at        | 2016-08-31T00:14:36                            |
+-------------------+------------------------------------------------+
   ```
### Add the router: external option to the provider network:

   `neutron net-update provider --router:external`  
   ```
   Updated network: provider
   ```
### Create and router and name it *"student-router"*
   `neutron router-create student01-router`
  ```  
Created a new router:
+-------------------------+--------------------------------------+
| Field                   | Value                                |
+-------------------------+--------------------------------------+
| admin_state_up          | True                                 |
| availability_zone_hints |                                      |
| availability_zones      |                                      |
| description             |                                      |
| distributed             | False                                |
| external_gateway_info   |                                      |
| ha                      | False                                |
| id                      | db95c14c-8a53-40ba-9e7d-292d210d4f8f |
| name                    | student01-router                     |
| routes                  |                                      |
| status                  | ACTIVE                               |
| tenant_id               | 810f65ae070847a2851d69344b1fad25     |
+-------------------------+--------------------------------------+

 ```

### Add the self-service network subnet as an interface on the router:

   `neutron router-interface-add student01-router student01`
   ```
Added interface 4347fd7a-543b-480a-a4cf-8fe1d53684ba to router student-router
   ```


### Set a gateway on the provider network on the router
   `neutron router-gateway-set student01-router provider`
   ```
   Set gateway for router student-router
   ```

### List network namespaces. You should see one qrouter namespace and two qdhcp namespaces

   `neutron router-port-list router`
  ```
 qrouter-db95c14c-8a53-40ba-9e7d-292d210d4f8f (id: 2)
 qdhcp-87361f8f-cad4-477e-81d1-7b9b8e2cc4d9 (id: 1)
 qdhcp-aed6e80e-f6c3-42c2-ad89-0d06c188d30b (id: 0)
  ```
 
`neutron router-port-list student01-router`  

   ```
   +--------------------------------------+------+-------------------+-------------------------------------------------------------------------------------+
| id                                   | name | mac_address       | fixed_ips                                                                           |
+--------------------------------------+------+-------------------+-------------------------------------------------------------------------------------+
| 71d216c7-492a-4d4c-9cd7-165b83be3cbe |      | fa:16:3e:c6:ed:6b | {"subnet_id": "998ec3dc-6497-46d8-ad7e-7e7dd73340b6", "ip_address": "172.16.1.1"}   |
| f61381ac-3f1c-4941-8633-57981952f9b0 |      | fa:16:3e:71:42:e7 | {"subnet_id": "25e05135-9d21-436c-8934-a670da127119", "ip_address": "10.3.200.103"} |
+--------------------------------------+------+-------------------+-------------------------------------------------------------------------------------+
  ```   
### Ping test for connectivity

Ping the 10.3.200.x address and test for connectivity
```
PING 10.3.200.103 (10.3.200.103) 56(84) bytes of data.
64 bytes from 10.3.200.103: icmp_seq=1 ttl=64 time=0.059 ms
64 bytes from 10.3.200.103: icmp_seq=2 ttl=64 time=0.056 ms
64 bytes from 10.3.200.103: icmp_seq=3 ttl=64 time=0.033 ms
^C
--- 10.3.200.103 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 1998ms
rtt min/avg/max/mdev = 0.033/0.049/0.059/0.012 ms
```