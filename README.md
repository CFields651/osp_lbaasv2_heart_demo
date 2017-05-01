# osp_lbaasv2_heat_demo  
This repo contains heat templates that provision a tenant network/subnet/router and instances.  
It also creates a loadbalancer/listener/pool with the instances as members and a health monitor. 
The cirros instances get a make-believe but functional web server that respond to VIP requests.  

#To create full stack:  
openstack stack create -t my_tenant_lbaas_master.yaml lbaas_demo -e my_tenant_params.yaml -e my_tenant_lbaas_registry.yaml   
#Once the stack is deployed successfully,   
fip=$(neutron floatingip-list | \  
  grep $(neutron lbaas-loadbalancer-show  -c vip_address -f value lbaas_demo_loadbalancer) | \  
  awk '{ print $6 }')  

#To demonstrate the fip working:  
curl $fip  

Note that you can adjust the number of instances with the num_instances param in my_tenant_params.yaml -  
but this does not apply to my_tenant_lbaasv2_stdalone.yaml.

#To investigate the lbaas use these commands to get started:   
   neutron lbaas-loadbalancer-list  
   neutron lbaas-loadbalancer-show [name of load blancer from above]   
   neutron lbaas-pool-list  
   neutron lbaas-pool-show stdalone_[name of pool from above]   
   neutron lbaas-member-list [name of pool from above]  
   neutron lbaas-listener-show [listener is from lbaas-pool-show]  

#Templates
my_tenant_lbaas_master.yaml: Master template that puts together a network, router, loadbalancer and instances  

Custom resource types or child templates:  
my_tenant_network.yaml: Provisions a tenant network, subnet and router  
my_tenant_InstanceWithFipAndCinderVolumeAndPoolMembership.yaml: Instance with make-believe web server and pool member  
my_tenant_lbaasv2_scaleup.yaml: Provisions a loadbalancer, Listener, Pool, Members and Healthmonitor  
my_tenant_lbaas_registry.yaml:Resource registry to define custom resource types  
my_tenant_params.yaml: Put the values for your parameters here  

my_tenant_lbaasv2_stdalone.yaml: This heat template creates a loadbalancer/listener/pool/members/healthmonitor from already existing tenant resoruces.  It is intended to be a simpler example than my_tenant_lbaas_scaleup.yaml  

#To create stand alone (loadblancer only) with my_tenant_lbaasv2_stdalone.yaml: 
Note: you will need an existing private subnet and two instances with local ips on that subnet for this template. 
openstack stack create -t  my_tenant_lbaasv2_stdalone.yaml stdalone_lbaasv2 --parameter "lb_subnet=[guid of existing subnet]" --parameter "member_ip1=[local ip of 1st instance]" --parameter "member_ip2=[local ip of 2nd instance]"  

Note: parameters could be specified this way too:  
--parameter "lb_subnet=c31cba83-4984-4e35-9cf4-56bc8577ff32;member_ip1=10.0.1.11;member_ip2=10.0.1.12"  
