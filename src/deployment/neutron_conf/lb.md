### Neutron LBaaS Agent

#### 需要注意的参数

1. `service_provider`。Neutron Server 端的类型驱动。

2. `device_driver`。LBaaS Agent 端的驱动。


#### 启动进程

```
/usr/bin/python2 /usr/bin/neutron-lbaasv2-agent \
--config-file /usr/share/neutron/neutron-dist.conf \
--config-file /usr/share/neutron/neutron-lbaas-dist.conf \
--config-file /etc/neutron/neutron.conf \
--config-file /etc/neutron/lbaas_agent.ini \
--config-dir /etc/neutron/conf.d/common \
--config-dir /etc/neutron/conf.d/neutron-lbaasv2-agent \
--log-file /var/log/neutron/lbaas-agent.log
```

#### neutron\_lbaas.conf

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

### services\_lbaas.conf

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

#### lbaas\_agent.ini

```
## From neutron.lbaas.agent#
# Seconds between periodic task runs (integer value)#periodic_interval = 10
# Drivers used to manage loadbalancing devices (multi valued)
#device_driver = neutron_lbaas.services.loadbalancer.drivers.haproxy.namespace_driver.HaproxyNSDriver

# Name of Open vSwitch bridge to use (string value)
#ovs_integration_bridge = br-int

# Uses veth for an OVS interface or not. Support kernels with limited namespace# support (e.g. RHEL 6.5) so long as ovs_use_veth is set to True. (boolean# value)#ovs_use_veth = false
# The driver used to manage the virtual interface. (string value)
#interface_driver = <None>

## From oslo.log#
# If set to true, the logging level will be set to DEBUG instead of the default
# INFO level. (boolean value)# Note: This option can be changed without restarting.
#debug = false

# DEPRECATED: If set to false, the logging level will be set to WARNING instead
# of the default INFO level. (boolean value)# This option is deprecated for removal.
# Its value may be silently ignored in the future.
#verbose = true

# The name of a logging configuration file. This file is appended to any
# existing logging configuration files. For details about logging configuration
# files, see the Python logging module documentation. Note that when logging
# configuration files are used then all logging configuration is set in the
# configuration file and other logging configuration options are ignored (for
# example, logging_context_format_string). (string value)
# Note: This option can be changed without restarting.
# Deprecated group/name - [DEFAULT]/log_config
#log_config_append = <None>

# Defines the format string for %%(asctime)s in log records. Default:
# %(default)s . This option is ignored if log_config_append is set. (string
# value)
#log_date_format = %Y-%m-%d %H:%M:%S

# (Optional) Name of log file to send logging output to. If no default is set,
# logging will go to stderr as defined by use_stderr. This option is ignored if
# log_config_append is set. (string value)
# Deprecated group/name - [DEFAULT]/logfile
#log_file = <None>

# (Optional) The base directory used for relative log_file paths. This option
# is ignored if log_config_append is set. (string value)
# Deprecated group/name - [DEFAULT]/logdir
#log_dir = <None>

# Uses logging handler designed to watch file system. When log file is moved or
# removed this handler will open a new log file with specified path
# instantaneously. It makes sense only if log_file option is specified and
# Linux platform is used. This option is ignored if log_config_append is set.
# (boolean value)
#watch_log_file = false

# Use syslog for logging. Existing syslog format is DEPRECATED and will be
# changed later to honor RFC5424. This option is ignored if log_config_append
# is set. (boolean value)
#use_syslog = false

# Syslog facility to receive log lines. This option is ignored if
# log_config_append is set. (string value)
#syslog_log_facility = LOG_USER

# Log output to standard error. This option is ignored if log_config_append is
# set. (boolean value)
#use_stderr = true

# Format string to use for log messages with context. (string value)
#logging_context_format_string = %(asctime)s.%(msecs)03d %(process)d %(levelname)s %(name)s [%(request_id)s %(user_identity)s] %(instance)s%(message)s

# Format string to use for log messages when context is undefined. (string
# value)
#logging_default_format_string = %(asctime)s.%(msecs)03d %(process)d %(levelname)s %(name)s [-] %(instance)s%(message)s

# Additional data to append to log message when logging level for the message
# is DEBUG. (string value)
#logging_debug_format_suffix = %(funcName)s %(pathname)s:%(lineno)d

# Prefix each line of exception output with this format. (string value)
#logging_exception_prefix = %(asctime)s.%(msecs)03d %(process)d ERROR %(name)s %(instance)s

# Defines the format string for %(user_identity)s that is used in
# logging_context_format_string. (string value)#logging_user_identity_format = %(user)s %(tenant)s %(domain)s %(user_domain)s %(project_domain)s

# List of package logging levels in logger=LEVEL pairs. This option is ignored
# if log_config_append is set. (list value)
#default_log_levels = amqp=WARN,amqplib=WARN,boto=WARN,qpid=WARN,sqlalchemy=WARN,suds=INFO,oslo.messaging=INFO,iso8601=WARN,requests.packages.urllib3.connectionpool=WARN,urllib3.connectionpool=WARN,websocket=WARN,requests.packages.urllib3.util.retry=WARN,urllib3.util.retry=WARN,keystonemiddleware=WARN,routes.middleware=WARN,stevedore=WARN,taskflow=WARN,keystoneauth=WARN,oslo.cache=INFO,dogpile.core.dogpile=INFO

# Enables or disables publication of error events. (boolean value)
#publish_errors = false

# The format for an instance that is passed with the log message. (string
# value)#instance_format = "[instance: %(uuid)s] "
# The format for an instance UUID that is passed with the log message. (string
# value)
#instance_uuid_format = "[instance: %(uuid)s] "

# Enables or disables fatal status of deprecations. (boolean value)
#fatal_deprecations = false

[haproxy]
## From neutron.lbaas.agent#
# Location to store config and state files (string value)
# Deprecated group/name - [DEFAULT]/loadbalancer_state_path
#loadbalancer_state_path = $state_path/lbaas

# The user group (string value)
# Deprecated group/name - [DEFAULT]/user_group
#user_group = nogroup

# When delete and re-add the same vip, send this many gratuitous ARPs to flush
# the ARP cache in the Router. Set it below or equal to 0 to disable this# feature. (integer value)
#send_gratuitous_arp = 3
```

