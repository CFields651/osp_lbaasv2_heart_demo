#lbaasv2-heat-demo  
This repo contains heat templates that provision a tenant network/subnet/router and instances.  
It also creates a loadbalancer/listener/pool with the instances as members and a health monitor. 
The cirros instances get a make-believe but functional web server that respond to VIP requests.  

#To create:  
openstack stack create -t my_tenant_lbaas_master.yaml lbaas_demo -e my_tenant_params.yaml -e my_tenant_lbaas_registry.yaml   

#Once the stack is deploy successfully,   
fip=$(neutron floatingip-list | \  
  grep $(neutron lbaas-loadbalancer-show  -c vip_address -f value lbaas_demo_loadbalancer) | \  
  awk '{ print $6 }')  

#To demonstrate the fip working:  
curl $fip  

Note that you can adjust the number of instances with the num_instances param in my_tenant_params.yaml  

my_tenant_lbaas_master.yaml  
|   Master template that puts together a network, router, loadbalancer and instances  
| 
--my_tenant_network.yaml  
    Provisions a tenant network, subnet and router  
|
--my_tenant_InstanceWithFipAndCinderVolumeAndPoolMembership.yaml  
|   Provision and instance with a FIP and a Cinder Volume and make-believe we server   
|
--my_tenant_lbaasv2_scaleup.yaml  
    Provisioning a loadbalancer, Listener, Pool, Members and Healthermonitor  
 
my_tenant_lbaasv2_stdalone.yaml  
  This heat template creates a loadbalancer/listener/pool/members/healthmonitor from already existing tenant resoruces.  It is intended to be a simpler example than my_tenant_lbaas_scaleup.yaml  

