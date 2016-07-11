### neutron_lbaas.conf

```
[DEFAULT]
# =========== items for agent scheduler extension =============
# loadbalancer_pool_scheduler_driver = neutron.services.loadbalancer.agent_scheduler.ChanceScheduler
# loadbalancer_pool_scheduler_driver = neutron.services.loadbalancer.agent_scheduler.LeastPoolAgentScheduler
# loadbalancer_scheduler_driver = neutron.agent_scheduler.ChanceScheduler

[quotas]
# Number of vips allowed per tenant. A negative value means unlimited.  This
# is only applicable when v1 of the lbaas extension is used.
# quota_vip = 10

# Number of pools allowed per tenant. A negative value means unlimited.  This
# is applicable in both the v1 and v2 lbaas extensions.
# quota_pool = 10

# Number of pool members allowed per tenant. A negative value means unlimited.
# The default is unlimited because a member is not a real resource consumer
# on Openstack. However, on back-end, a member is a resource consumer
# and that is the reason why quota is possible.  This is applicable in both
# the v1 and v2 lbaas extensions.
# quota_member = -1

# Number of health monitors allowed per tenant. A negative value means
# unlimited.
# The default is unlimited because a health monitor is not a real resource
# consumer on Openstack. However, on back-end, a health monitor is a resource
# consumer and that is the reason why quota is possible.  This is only
# applicable when using the v1 lbaas extension.  The quota for the v2 lbaas
# extension health monitor is quota_healthmonitor.
# quota_health_monitor = -1

# Number of loadbalancers allowed per tenant. A negative value means unlimited.
# This is only applicable in the v2 lbaas extension.
# quota_loadbalancer = 10

# Number of listeners allowed per tenant. A negative value means unlimited.
# This is only applicable in the v2 lbaas extension.
# quota_listener = -1

# Number of v2 health monitors allowed per tenant. A negative value means
# unlimited. This is only applicable in the v2 lbaas extension.  The quota for
# the v1 lbaas extension health monitor is configured with
# quota_health_monitor.
# quota_healthmonitor = -1

[service_auth]
# auth_url = http://127.0.0.1:5000/v2.0
# admin_tenant_name = %SERVICE_TENANT_NAME%
# admin_user = %SERVICE_USER%
# admin_password = %SERVICE_PASSWORD%
# admin_user_domain = %SERVICE_USER_DOMAIN%
# admin_project_domain = %SERVICE_PROJECT_DOMAIN%
# region = %REGION%
# service_name = lbaas
# auth_version = 2

[service_providers]
# Must be in form:
# service_provider=<service_type>:<name>:<driver>[:default]
# List of allowed service types includes LOADBALANCER
# Combination of <service type> and <name> must be unique; <driver> must also be unique
# This is multiline option
# service_provider=LOADBALANCER:name:lbaas_plugin_driver_path:default
# service_provider=LOADBALANCER:Haproxy:neutron_lbaas.services.loadbalancer.drivers.haproxy.plugin_driver.HaproxyOnHostPluginDriver:default
# service_provider = LOADBALANCER:radware:neutron_lbaas.services.loadbalancer.drivers.radware.driver.LoadBalancerDriver:default
# service_provider = LOADBALANCERV2:radwarev2:neutron_lbaas.drivers.radware.v2_driver.RadwareLBaaSV2Driver:default
# service_provider=LOADBALANCER:NetScaler:neutron_lbaas.services.loadbalancer.drivers.netscaler.netscaler_driver.NetScalerPluginDriver
# service_provider=LOADBALANCER:Embrane:neutron_lbaas.services.loadbalancer.drivers.embrane.driver.EmbraneLbaas:default
# service_provider = LOADBALANCER:A10Networks:neutron_lbaas.services.loadbalancer.drivers.a10networks.driver_v1.ThunderDriver:default
# service_provider = LOADBALANCER:VMWareEdge:neutron_lbaas.services.loadbalancer.drivers.vmware.edge_driver.EdgeLoadbalancerDriver:default

# LBaaS v2 drivers
#service_provider = LOADBALANCERV2:Octavia:neutron_lbaas.drivers.octavia.driver.OctaviaDriver:default
# service_provider = LOADBALANCERV2:LoggingNoop:neutron_lbaas.drivers.logging_noop.driver.LoggingNoopLoadBalancerDriver:default
service_provider=LOADBALANCERV2:Haproxy:neutron_lbaas.drivers.haproxy.plugin_driver.HaproxyOnHostPluginDriver:default
# service_provider = LOADBALANCERV2:A10Networks:neutron_lbaas.drivers.a10networks.driver_v2.ThunderDriver:default
# service_provider = LOADBALANCERV2:brocade:neutron_lbaas.drivers.brocade.driver_v2.BrocadeLoadBalancerDriver:default
# service_provider = LOADBALANCERV2:kemptechnologies:neutron_lbaas.drivers.kemptechnologies.driver_v2.KempLoadMasterDriver:default

[certificates]
# cert_manager_type = barbican
## The following option is only valid when using neutron_lbaas.common.cert_manager.local_cert_manager
# storage_path = /var/lib/neutron-lbaas/certificates/
```

### services_lbaas.conf
```
[radware]
#vdirect_address = 0.0.0.0
#ha_secondary_address=
#vdirect_user = vDirect
#vdirect_password = radware
#service_ha_pair = False
#service_throughput = 1000
#service_ssl_throughput = 200
#service_compression_throughput = 100
#service_cache = 20
#service_adc_type = VA
#service_adc_version=
#service_session_mirroring_enabled = False
#service_isl_vlan = -1
#service_resource_pool_ids = []
#actions_to_skip = 'setup_l2_l3'
#l4_action_name = 'BaseCreate'
#l2_l3_workflow_name = openstack_l2_l3
#l4_workflow_name = openstack_l4
#l2_l3_ctor_params = service: _REPLACE_, ha_network_name: HA-Network, ha_ip_pool_name: default, allocate_ha_vrrp: True, allocate_ha_ips: True
#l2_l3_setup_params = data_port: 1, data_ip_address: 192.168.200.99, data_ip_mask: 255.255.255.0, gateway: 192.168.200.1, ha_port: 2

[radwarev2]
#vdirect_address = 0.0.0.0
#ha_secondary_address=
#vdirect_user = vDirect
#vdirect_password = radware
#service_ha_pair = False
#service_throughput = 1000
#service_ssl_throughput = 200
#service_compression_throughput = 100
#service_cache = 20
#service_adc_type = VA
#service_adc_version=
#service_session_mirroring_enabled = False
#service_isl_vlan = -1
#service_resource_pool_ids = []
#workflow_template_name = os_lb_v2
#child_workflow_template_names = [manage_l3]
#workflow_params = twoleg_enabled: _REPLACE_, ha_network_name: HA-Network, ha_ip_pool_name: default, allocate_ha_vrrp: True, allocate_ha_ips: True, data_port: 1, data_ip_address: 192.168.200.99, data_ip_mask: 255.255.255.0, gateway: 192.168.200.1, ha_port: 2"
#workflow_action_name = apply
#stats_action_name = stats

[radwarev2_debug]
#provision_service = True
#configure_l3 = True
#configure_l4 = True

[netscaler_driver]
#netscaler_ncc_uri = https://ncc_server.acme.org/ncc/v1/api
#netscaler_ncc_username = admin
#netscaler_ncc_password = secret

[heleoslb]
#esm_mgmt =
#admin_username =
#admin_password =
#lb_image =
#inband_id =
#oob_id =
#mgmt_id =
#dummy_utif_id =
#resource_pool_id =
#async_requests =
#lb_flavor = small
#sync_interval = 60

[haproxy]
#jinja_config_template = /opt/stack/neutron/neutron/services/drivers/haproxy/templates/haproxy_v1.4.template
#periodic_interval = 10
#interface_driver = neutron.agent.linux.interface.OVSInterfaceDriver
#send_gratuitous_arp = 3
#user_group = nogroup
#loadbalancer_state_path = $state_path/lbaas

[octavia]
# base_url = http://localhost:9191
```
