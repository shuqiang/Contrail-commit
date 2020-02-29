# NOTE: Refer this documentation to use contrail-ansible-deployer to deploy Contrail, then integration with OpenStack Ocata.


## 1 Using contrail-ansible-deploy to deploy opencontrail, run the following step 1.1 to step 1.3.

#### step 1.1,	install ansible and ansible deployer
```
yum install -y ansible-2.4.2.0 git

git clone -b R5.0 http://github.com/Juniper/contrail-ansible-deployer //notice the branch R5.0 
```
#### step 1.2,	configure instances.yaml file
Note: 
* ansible build network: 192.168.1.XX .
* control plane network: 172.16.0.1/24 .
* data plane network: 172.16.2.1/24 .

```
provider_config:
  bms:
    ssh_pwd: r00tme
    ssh_user: root
    ssh_public_key: /root/.ssh/id_rsa.pub
    ssh_private_key: /root/.ssh/id_rsa
    ntpserver: 192.168.1.254
instances:
  bms1:
    provider: bms
    ip: 172.16.0.101
    roles:
      config_database:
      config:
      control:
      webui:
  bms2:
    provider: bms
    ip: 172.16.0.102
    roles:
      config_database:
      config:
      control:
      webui:
  bms3:
    provider: bms
    ip: 172.16.0.103
    roles:
      config_database:
      config:
      control:
      webui:
  bms4:
    provider: bms
    ip: 172.16.0.104
    roles:
      analytics_database:
      analytics:
  bms5:
    provider: bms
    ip: 172.16.0.105
    roles:
      analytics_database:
      analytics:
  bms6:
    provider: bms
    ip: 172.16.0.106
    roles:
      analytics_database:
      analytics:
  bms7:
    provider: bms
    ip: 172.16.0.1
    roles:
      vrouter:
  bms8:
    provider: bms
    ip: 172.16.0.2
    roles:
      vrouter:
  bms9:
    provider: bms
    ip: 172.16.0.3
    roles:
      vrouter:
  bms10:
    provider: bms
    ip: 172.16.0.4
    roles:
      vrouter:
  bms11:
    provider: bms
    ip: 172.16.0.202
    roles:
      vrouter:
        TSN_EVPN_MODE: True
        TSN_NODES: 172.16.2.202
global_configuration:
  CONTAINER_REGISTRY: 192.168.1.100:5000
  REGISTRY_PRIVATE_INSECURE: True
contrail_configuration:
  CONTRAIL_VERSION: 5.0.3-0.493-queens
  CLOUD_ORCHESTRATOR: openstack
  CONTROLLER_NODES: 172.16.0.101,172.16.0.102,172.16.0.103
  CONTROL_NODES: 172.16.0.101,172.16.0.102,172.16.0.103
  CONFIGDB_NODES: 172.16.0.101,172.16.0.102,172.16.0.103
  ANALYTICS_NODES: 172.16.0.104,172.16.0.105,172.16.0.106
  ANALYTICSDB_NODES: 172.16.0.104,172.16.0.105,172.16.0.106
  VROUTER_GATEWAY: 172.16.2.254
  PHYSICAL_INTERFACE: eth2
  AAA_MODE: cloud-admin
  AUTH_MODE: keystone
  KEYSTONE_AUTH_ADMIN_TENANT: admin
  KEYSTONE_AUTH_ADMIN_USER: admin
  KEYSTONE_AUTH_ADMIN_PASSWORD: oss!SDS@openstack20190530
  KEYSTONE_AUTH_HOST: 172.16.0.254
  KEYSTONE_AUTH_URL_VERSION: '/v3'
  KEYSTONE_AUTH_URL_TOKENS: '/v3/auth/tokens'
  KEYSTONE_AUTH_USER_DOMAIN_NAME: default
  KEYSTONE_AUTH_PROJECT_DOMAIN_NAME: default
  CONFIG_DATABASE_NODEMGR__DEFAULTS__minimum_diskGB: 4
  DATABASE_NODEMGR__DEFAULTS__minimum_diskGB: 4
  METADATA_PROXY_SECRET: v3YH6KeEkyhVJMa2glm25XkEkvnEr4qbkZpb5n9N
  BGP_ASN: 64513
kolla_config:
  kolla_globals:
    kolla_internal_vip_address: 172.16.0.254
    kolla_external_vip_address: 172.16.0.254
    contrail_api_interface_address: 172.16.0.254

  kolla_passwords:
    keystone_admin_password: contrail123
```

#### step 1.3,	start to deploy
```
ansible-playbook -e orchestrator=openstack -i inventory/ playbooks/configure_instances.yml
ansible-playbook -e orchestrator=openstack -i inventory/ playbooks/install_contrail.yml
```
Note：Not process the step: ansible-playbook -i inventory/ playbooks/install_openstack.yml
The Openstack has been intsalled by Kola.

## 2 Integration with Ocata
As the Openstack was intsalled by Kolla, we need to do these things to integrate with it:
  neutron and compute nodes install rpm packages；modify /etc/neutron/neutron.conf to add core_plugin，ContrailPlugin.ini add apiserver keystone info; compute nodes stop ovs and install vrouter-port-control.

### 2.1 Control nodes steps
It's include all the neutron-server nodes，run the following step 2.1.1 to step 2.1.4.

#### step 2.1.1, configure /etc/neutron/neutron.conf
file contents：
```
[DEFAULT]
core_plugin = neutron_plugin_contrail.plugins.opencontrail.contrail_plugin.NeutronPluginContrailCoreV2
service_plugins = neutron_plugin_contrail.plugins.opencontrail.loadbalancer.v2.plugin.LoadBalancerPluginV2
api_extensions_path = /usr/lib/python2.7/site-packages/neutron_plugin_contrail/extensions:/usr/lib/python2.7/site-packages/neutron_lbaas/extensions

[quotas]
quota_network = -1
quota_subnet = -1
quota_port = -1
quota_driver = neutron_plugin_contrail.plugins.opencontrail.quota.driver.QuotaDriver
```
#### step 2.1.2,  add ContrailPlugin.ini
file name：/etc/neutron/plugins/opencontrail/ContrailPlugin.ini

file contents：
```
[APISERVER]
api_server_ip = 10.0.0.254
api_server_port = 8082
multi_tenancy = True
contrail_extensions = ipam:neutron_plugin_contrail.plugins.opencontrail.contrail_plugin_ipam.NeutronPluginContrailIpam,policy:neutron_plugin_contrail.plugins.opencontrail.contrail_plugin_policy.NeutronPluginContrailPolicy,route-table:neutron_plugin_contrail.plugins.opencontrail.contrail_plugin_vpc.NeutronPluginContrailVpc,contrail:None,service-interface:None,vf-binding:None

[COLLECTOR]
analytics_api_ip = 10.0.0.254
analytics_api_port = 8081
```

#### step 2.1.3, copy the neutron init python scripts
```
copy /root/contrail-openstack-neutron-init-queens/* to  {#Neutron Containers}/usr/lib/python2.7/site-packages/ 
```
#### step 2.1.4, update configure file for /usr/bin/neutron-server 
```
use --config-file /etc/neutron/plugins/opencontrail/ContrailPlugin.ini 
replace --config-file /etc/neutron/plugins/ml2/ml2_conf.ini
```
### 2.2 Compute nodes steps, stop ovs and install compute plugin for contrail, run the following step 2.2.1 to step 2.2.2

#### step 2.2.1, stop ovs
```
docker exec -it openvswitch_vswitchd bash

#execute in containers
ovs-dpctl dump-dps
ovs-dpctl del-dp system@ovs-system

docker stop neutron_openvswitch_agent openvswitch_vswitchd openvswitch_db
rmmod vport_vxlan
rmmod openvswitch

# the ovs container will grab the NIC for vhost0, delete it
docker rm neutron_openvswitch_agent openvswitch_vswitchd openvswitch_db

ip link delete qbr
ip link delete qvo
```

#### step 2.2.2, install compute plugin
```
tar -zxvf compute-plugin.tgz
docker cp contrail-openstack-compute-init-queens/bin/vrouter-port-control nova_compute:/usr/bin/
docker cp contrail-openstack-compute-init-queens/site-packages/nova_contrail_vif nova_libvirt:/usr/lib/python2.7/site-packages/
docker cp contrail-openstack-compute-init-queens/site-packages/nova_contrail_vif-0.1-py2.7.egg-info/ nova_libvirt:/usr/lib/python2.7/site-packages/
docker cp contrail-openstack-compute-init-queens/site-packages/vif_plug_vrouter/ nova_libvirt:/usr/lib/python2.7/site-packages/
#docker cp contrail-openstack-compute-init-queens/bin/vrouter-port-control nova_libvirt:/usr/bin/
docker restart nova_compute
docker restart nova_libvirt
```

