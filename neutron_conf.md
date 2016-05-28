## Neutron 配置

### 概述

Neutron 配置主要涉及：
```
├── dhcp_agent.ini
├── dnsmasq-neutron.conf
├── l3_agent.ini
├── lbaas_agent.ini
├── metadata_agent.ini
├── metering_agent.ini
├── neutron.conf
├── neutron_lbaas.conf
├── neutron_vpnaas.conf
├── plugin.ini -> /etc/neutron/plugins/ml2/ml2_conf.ini
├── plugins
│   └── ml2
│       ├── ml2_conf.ini
│       └── openvswitch_agent.ini
├── policy.json
├── rootwrap.conf
├── services_lbaas.conf
└── vpn_agent.ini
```

各作用解释

### Neutron.conf

```
[DEFAULT]
# Print more verbose output (set logging level to INFO instead of default WARNING level).
# verbose = True
verbose = True

# =========Start Global Config Option for Distributed L3 Router===============
# Setting the "router_distributed" flag to "True" will default to the creation
# of distributed tenant routers. The admin can override this flag by specifying
# the type of the router on the create request (admin-only attribute). Default
# value is "False" to support legacy mode (centralized) routers.
#
# router_distributed = False
router_distributed = True
#
# ===========End Global Config Option for Distributed L3 Router===============

# Print debugging output (set logging level to DEBUG instead of default WARNING level).
# debug = False
debug = False

# Where to store Neutron state files.  This directory must be writable by the
# user executing the agent.
# state_path = /var/lib/neutron
state_path = /var/lib/neutron

# log_format = %(asctime)s %(levelname)8s [%(name)s] %(message)s
# log_date_format = %Y-%m-%d %H:%M:%S

# use_syslog                           -> syslog
# log_file and log_dir                 -> log_dir/log_file
# (not log_file) and log_dir           -> log_dir/{binary_name}.log
# use_stderr                           -> stderr
# (not user_stderr) and (not log_file) -> stdout
# publish_errors                       -> notification system

# use_syslog = False
use_syslog = False
# syslog_log_facility = LOG_USER

# use_stderr = False
use_stderr = True
# log_file =
# log_dir =
log_dir =/var/log/neutron

# publish_errors = False

# Address to bind the API server to
# bind_host = 0.0.0.0
bind_host = 0.0.0.0

# Port the bind the API server to
# bind_port = 9696
bind_port = 9696

# Path to the extensions.  Note that this can be a colon-separated list of
# paths.  For example:
# api_extensions_path = extensions:/path/to/more/extensions:/even/more/extensions
# The __path__ of neutron.extensions is appended to this, so if your
# extensions are in there you don't need to specify them here
# api_extensions_path =

# (StrOpt) Neutron core plugin entrypoint to be loaded from the
# neutron.core_plugins namespace. See setup.cfg for the entrypoint names of the
# plugins included in the neutron source distribution. For compatibility with
# previous versions, the class name of a plugin can be specified instead of its
# entrypoint name.
#
# core_plugin =
core_plugin =ml2
# Example: core_plugin = ml2

# (StrOpt) Neutron IPAM (IP address management) driver to be loaded from the
# neutron.ipam_drivers namespace. See setup.cfg for the entry point names.
# If ipam_driver is not set (default behavior), no ipam driver is used.
# Example: ipam_driver =
# In order to use the reference implementation of neutron ipam driver, use
# 'internal'.
# Example: ipam_driver = internal

# (ListOpt) List of service plugin entrypoints to be loaded from the
# neutron.service_plugins namespace. See setup.cfg for the entrypoint names of
# the plugins included in the neutron source distribution. For compatibility
# with previous versions, the class name of a plugin can be specified instead
# of its entrypoint name.
#
# service_plugins =
service_plugins =router,vpnaas,neutron_lbaas.services.loadbalancer.plugin.LoadBalancerPluginv2
# Example: service_plugins = router,firewall,lbaas,vpnaas,metering,qos

# Paste configuration file
# api_paste_config = /usr/share/neutron/api-paste.ini
api_paste_config = /usr/share/neutron/api-paste.ini

# (StrOpt) Hostname to be used by the neutron server, agents and services
# running on this machine. All the agents and services running on this machine
# must use the same host value.
# The default value is hostname of the machine.
#
# host =

# The strategy to be used for auth.
# Supported values are 'keystone'(default), 'noauth'.
# auth_strategy = noauth
auth_strategy = keystone

# Base MAC address. The first 3 octets will remain unchanged. If the
# 4h octet is not 00, it will also be used. The others will be
# randomly generated.
# 3 octet
# base_mac = fa:16:3e:00:00:00
base_mac = fa:16:3e:00:00:00
# 4 octet
# base_mac = fa:16:3e:4f:00:00

# DVR Base MAC address. The first 3 octets will remain unchanged. If the
# 4th octet is not 00, it will also be used.  The others will be randomly
# generated. The 'dvr_base_mac' *must* be different from 'base_mac' to
# avoid mixing them up with MAC's allocated for tenant ports.
# A 4 octet example would be dvr_base_mac = fa:16:3f:4f:00:00
# The default is 3 octet
# dvr_base_mac = fa:16:3f:00:00:00

# Maximum amount of retries to generate a unique MAC address
# mac_generation_retries = 16
mac_generation_retries = 16

# DHCP Lease duration (in seconds).  Use -1 to
# tell dnsmasq to use infinite lease times.
# dhcp_lease_duration = 86400
dhcp_lease_duration = 86400

# Domain to use for building the hostnames
# dns_domain = openstacklocal

# Allow sending resource operation notification to DHCP agent
# dhcp_agent_notification = True
dhcp_agent_notification = True

# Enable or disable bulk create/update/delete operations
# allow_bulk = True
allow_bulk = True
# Enable or disable pagination
# allow_pagination = False
allow_pagination = False
# Enable or disable sorting
# allow_sorting = False
allow_sorting = False
# Enable or disable overlapping IPs for subnets
# Attention: the following parameter MUST be set to False if Neutron is
# being used in conjunction with nova security groups
# allow_overlapping_ips = True
allow_overlapping_ips = True
# Ensure that configured gateway is on subnet. For IPv6, validate only if
# gateway is not a link local address. Deprecated, to be removed during the
# K release, at which point the check will be mandatory.
# force_gateway_on_subnet = True

# Default maximum number of items returned in a single response,
# value == infinite and value < 0 means no max limit, and value must
# be greater than 0. If the number of items requested is greater than
# pagination_max_limit, server will just return pagination_max_limit
# of number of items.
# pagination_max_limit = -1

# Maximum number of DNS nameservers per subnet
# max_dns_nameservers = 5

# Maximum number of host routes per subnet
# max_subnet_host_routes = 20

# Maximum number of fixed ips per port
# max_fixed_ips_per_port = 5

# Maximum number of routes per router
# max_routes = 30

# Default Subnet Pool to be used for IPv4 subnet-allocation.
# Specifies by UUID the pool to be used in case of subnet-create being called
# without a subnet-pool ID.  The default of None means that no pool will be
# used unless passed explicitly to subnet create.  If no pool is used, then a
# CIDR must be passed to create a subnet and that subnet will not be allocated
# from any pool; it will be considered part of the tenant's private address
# space.
# default_ipv4_subnet_pool =

# Default Subnet Pool to be used for IPv6 subnet-allocation.
# Specifies by UUID the pool to be used in case of subnet-create being
# called without a subnet-pool ID.  Set to "prefix_delegation"
# to enable IPv6 Prefix Delegation in a PD-capable environment.
# See the description for default_ipv4_subnet_pool for more information.
# default_ipv6_subnet_pool =

# =========== items for MTU selection and advertisement =============
# Advertise MTU.  If True, effort is made to advertise MTU
# settings to VMs via network methods (ie. DHCP and RA MTU options)
# when the network's preferred MTU is known.
# advertise_mtu = False
advertise_mtu = False
# ======== end of items for MTU selection and advertisement =========

# =========== items for agent management extension =============
# Seconds to regard the agent as down; should be at least twice
# report_interval, to be sure the agent is down for good
# agent_down_time = 75
agent_down_time = 75

# Agent starts with admin_state_up=False when enable_new_agents=False.
# In the case, user's resources will not be scheduled automatically to the
# agent until admin changes admin_state_up to True.
# enable_new_agents = True
# ===========  end of items for agent management extension =====

# =========== items for agent scheduler extension =============
# Driver to use for scheduling network to DHCP agent
network_scheduler_driver = neutron.scheduler.dhcp_agent_scheduler.WeightScheduler
# Driver to use for scheduling router to a default L3 agent
# router_scheduler_driver = neutron.scheduler.l3_agent_scheduler.LeastRoutersScheduler
router_scheduler_driver = neutron.scheduler.l3_agent_scheduler.ChanceScheduler
# Driver to use for scheduling a loadbalancer pool to an lbaas agent
# loadbalancer_pool_scheduler_driver = neutron.services.loadbalancer.agent_scheduler.ChanceScheduler

# (StrOpt) Representing the resource type whose load is being reported by
# the agent.
# This can be 'networks','subnets' or 'ports'. When specified (Default is networks),
# the server will extract particular load sent as part of its agent configuration object
# from the agent report state, which is the number of resources being consumed, at
# every report_interval.
# dhcp_load_type can be used in combination with network_scheduler_driver =
# neutron.scheduler.dhcp_agent_scheduler.WeightScheduler
# When the network_scheduler_driver is WeightScheduler, dhcp_load_type can
# be configured to represent the choice for the resource being balanced.
# Example: dhcp_load_type = networks
# Values:
#   networks - number of networks hosted on the agent
#   subnets -  number of subnets associated with the networks hosted on the agent
#   ports   -  number of ports associated with the networks hosted on the agent
dhcp_load_type = networks

# Allow auto scheduling networks to DHCP agent. It will schedule non-hosted
# networks to first DHCP agent which sends get_active_networks message to
# neutron server
# network_auto_schedule = True

# Allow auto scheduling routers to L3 agent. It will schedule non-hosted
# routers to first L3 agent which sends sync_routers message to neutron server
# router_auto_schedule = True

# Allow automatic rescheduling of routers from dead L3 agents with
# admin_state_up set to True to alive agents.
# allow_automatic_l3agent_failover = False
allow_automatic_l3agent_failover = False

# Allow automatic removal of networks from dead DHCP agents with
# admin_state_up set to True.
# Networks could then be rescheduled if network_auto_schedule is True
# allow_automatic_dhcp_failover = True

# Number of DHCP agents scheduled to host a tenant network.
# If this number is greater than 1, the scheduler automatically
# assigns multiple DHCP agents for a given tenant network,
# providing high availability for DHCP service.
# dhcp_agents_per_network = 1
dhcp_agents_per_network = 2

# Enable services on agents with admin_state_up False.
# If this option is False, when admin_state_up of an agent is turned to
# False, services on it will be disabled. If this option is True, services
# on agents with admin_state_up False keep available and manual scheduling
# to such agents is available. Agents with admin_state_up False are not
# selected for automatic scheduling regardless of this option.
# enable_services_on_agents_with_admin_state_down = False

# ===========  end of items for agent scheduler extension =====

# =========== items for l3 extension ==============
# Enable high availability for virtual routers.
# l3_ha = False
l3_ha = False
#
# Maximum number of l3 agents which a HA router will be scheduled on. If it
# is set to 0 the router will be scheduled on every agent.
max_l3_agents_per_router = 2
#
# Minimum number of l3 agents which a HA router will be scheduled on. The
# default value is 2.
min_l3_agents_per_router = 2
#
# CIDR of the administrative network if HA mode is enabled
l3_ha_net_cidr = 169.254.192.0/18
#
# Enable snat by default on external gateway when available
# enable_snat_by_default = True
#
# The network type to use when creating the HA network for an HA router.
# By default or if empty, the first 'tenant_network_types'
# is used. This is helpful when the VRRP traffic should use a specific
# network which not the default one.
# ha_network_type =
# Example: ha_network_type = flat
#
# The physical network name with which the HA network can be created.
# ha_network_physical_name =
# Example: ha_network_physical_name = physnet1
# =========== end of items for l3 extension =======

# =========== items for metadata proxy configuration ==============
# User (uid or name) running metadata proxy after its initialization
# (if empty: agent effective user)
# metadata_proxy_user =

# Group (gid or name) running metadata proxy after its initialization
# (if empty: agent effective group)
# metadata_proxy_group =

# Enable/Disable log watch by metadata proxy, it should be disabled when
# metadata_proxy_user/group is not allowed to read/write its log file and
# 'copytruncate' logrotate option must be used if logrotate is enabled on
# metadata proxy log files. Option default value is deduced from
# metadata_proxy_user: watch log is enabled if metadata_proxy_user is agent
# effective user id/name.
# metadata_proxy_watch_log =

# Location of Metadata Proxy UNIX domain socket
# metadata_proxy_socket = $state_path/metadata_proxy
# =========== end of items for metadata proxy configuration ==============

# ========== items for VLAN trunking networks ==========
# Setting this flag to True will allow plugins that support it to
# create VLAN transparent networks. This flag has no effect for
# plugins that do not support VLAN transparent networks.
# vlan_transparent = False
# ========== end of items for VLAN trunking networks ==========

# =========== WSGI parameters related to the API server ==============
# Number of separate API worker processes to spawn. If not specified or < 1,
# the default value is equal to the number of CPUs available.
# api_workers = <number of CPUs>
api_workers = 4

# Number of separate RPC worker processes to spawn. If not specified or < 1,
# a single RPC worker process is spawned by the parent process.
# rpc_workers = 1
rpc_workers = 4

# Timeout for client connections socket operations. If an
# incoming connection is idle for this number of seconds it
# will be closed. A value of '0' means wait forever. (integer
# value)
# client_socket_timeout = 900

# wsgi keepalive option. Determines if connections are allowed to be held open
# by clients after a request is fulfilled. A value of False will ensure that
# the socket connection will be explicitly closed once a response has been
# sent to the client.
# wsgi_keep_alive = True

# Sets the value of TCP_KEEPIDLE in seconds to use for each server socket when
# starting API server. Not supported on OS X.
# tcp_keepidle = 600

# Number of seconds to keep retrying to listen
# retry_until_window = 30

# Number of backlog requests to configure the socket with.
# backlog = 4096

# Max header line to accommodate large tokens
# max_header_line = 16384

# Enable SSL on the API server
# use_ssl = False
use_ssl = False

# Certificate file to use when starting API server securely
# ssl_cert_file = /path/to/certfile

# Private key file to use when starting API server securely
# ssl_key_file = /path/to/keyfile

# CA certificate file to use when starting API server securely to
# verify connecting clients. This is an optional parameter only required if
# API clients need to authenticate to the API server using SSL certificates
# signed by a trusted CA
# ssl_ca_file = /path/to/cafile
# ======== end of WSGI parameters related to the API server ==========

# ======== neutron nova interactions ==========
# Send notification to nova when port status is active.
# notify_nova_on_port_status_changes = False
notify_nova_on_port_status_changes = True

# Send notifications to nova when port data (fixed_ips/floatingips) change
# so nova can update it's cache.
# notify_nova_on_port_data_changes = False
notify_nova_on_port_data_changes = True

# URL for connection to nova (Only supports one nova region currently).
# nova_url = http://127.0.0.1:8774/v2
nova_url = http://lb.137.saic.ustack.in:8774/v2

# Name of nova region to use. Useful if keystone manages more than one region
# nova_region_name =

# Username for connection to nova in admin context
# nova_admin_username =
nova_admin_username =nova

# The uuid of the admin nova tenant
# nova_admin_tenant_id =

# The name of the admin nova tenant. If the uuid of the admin nova tenant
# is set, this is optional.  Useful for cases where the uuid of the admin
# nova tenant is not available when configuration is being done.
# nova_admin_tenant_name =
nova_admin_tenant_name =services

# Password for connection to nova in admin context.
# nova_admin_password =
# nova_admin_password =cd2185ddf981f7af15d373d3
nova_admin_password =

# Authorization URL for connection to nova in admin context.
# nova_admin_auth_url =
nova_admin_auth_url =

# CA file for novaclient to verify server certificates
# nova_ca_certificates_file =

# Boolean to control ignoring SSL errors on the nova url
# nova_api_insecure = False

# Number of seconds between sending events to nova if there are any events to send
# send_events_interval = 2
send_events_interval = 2

# ======== end of neutron nova interactions ==========

#
# Options defined in oslo.messaging
#

# Use durable queues in amqp. (boolean value)
# Deprecated group/name - [DEFAULT]/rabbit_durable_queues
# amqp_durable_queues=false

# Auto-delete queues in amqp. (boolean value)
# amqp_auto_delete=false

# Size of RPC connection pool. (integer value)
# rpc_conn_pool_size=30

# Qpid broker hostname. (string value)
# qpid_hostname=localhost

# Qpid broker port. (integer value)
# qpid_port=5672

# Qpid HA cluster host:port pairs. (list value)
# qpid_hosts=$qpid_hostname:$qpid_port

# Username for Qpid connection. (string value)
# qpid_username=

# Password for Qpid connection. (string value)
# qpid_password=

# Space separated list of SASL mechanisms to use for auth.
# (string value)
# qpid_sasl_mechanisms=

# Seconds between connection keepalive heartbeats. (integer
# value)
# qpid_heartbeat=60

# Transport to use, either 'tcp' or 'ssl'. (string value)
# qpid_protocol=tcp

# Whether to disable the Nagle algorithm. (boolean value)
# qpid_tcp_nodelay=true

# The qpid topology version to use.  Version 1 is what was
# originally used by impl_qpid.  Version 2 includes some
# backwards-incompatible changes that allow broker federation
# to work.  Users should update to version 2 when they are
# able to take everything down, as it requires a clean break.
# (integer value)
# qpid_topology_version=1

# SSL version to use (valid only if SSL enabled). valid values
# are TLSv1, SSLv23 and SSLv3. SSLv2 may be available on some
# distributions. (string value)
# kombu_ssl_version=

# SSL key file (valid only if SSL enabled). (string value)
# kombu_ssl_keyfile=

# SSL cert file (valid only if SSL enabled). (string value)
# kombu_ssl_certfile=

# SSL certification authority file (valid only if SSL
# enabled). (string value)
# kombu_ssl_ca_certs=

# How long to wait before reconnecting in response to an AMQP
# consumer cancel notification. (floating point value)
# kombu_reconnect_delay=1.0

# The RabbitMQ broker address where a single node is used.
# (string value)
# rabbit_host=localhost

# The RabbitMQ broker port where a single node is used.
# (integer value)
# rabbit_port=5672

# RabbitMQ HA cluster host:port pairs. (list value)
# rabbit_hosts=$rabbit_host:$rabbit_port

# Connect over SSL for RabbitMQ. (boolean value)
# rabbit_use_ssl=false

# The RabbitMQ userid. (string value)
# rabbit_userid=guest

# The RabbitMQ password. (string value)
# rabbit_password=guest

# the RabbitMQ login method (string value)
# rabbit_login_method=AMQPLAIN

# The RabbitMQ virtual host. (string value)
# rabbit_virtual_host=/

# How frequently to retry connecting with RabbitMQ. (integer
# value)
# rabbit_retry_interval=1

# How long to backoff for between retries when connecting to
# RabbitMQ. (integer value)
# rabbit_retry_backoff=2

# Maximum number of RabbitMQ connection retries. Default is 0
# (infinite retry count). (integer value)
# rabbit_max_retries=0

# Use HA queues in RabbitMQ (x-ha-policy: all). If you change
# this option, you must wipe the RabbitMQ database. (boolean
# value)
# rabbit_ha_queues=false

# If passed, use a fake RabbitMQ provider. (boolean value)
# fake_rabbit=false

# ZeroMQ bind address. Should be a wildcard (*), an ethernet
# interface, or IP. The "host" option should point or resolve
# to this address. (string value)
# rpc_zmq_bind_address=*

# MatchMaker driver. (string value)
# rpc_zmq_matchmaker=oslo.messaging._drivers.matchmaker.MatchMakerLocalhost

# ZeroMQ receiver listening port. (integer value)
# rpc_zmq_port=9501

# Number of ZeroMQ contexts, defaults to 1. (integer value)
# rpc_zmq_contexts=1

# Maximum number of ingress messages to locally buffer per
# topic. Default is unlimited. (integer value)
# rpc_zmq_topic_backlog=

# Directory for holding IPC sockets. (string value)
# rpc_zmq_ipc_dir=/var/run/openstack

# Name of this node. Must be a valid hostname, FQDN, or IP
# address. Must match "host" option, if running Nova. (string
# value)
# rpc_zmq_host=oslo

# Seconds to wait before a cast expires (TTL). Only supported
# by impl_zmq. (integer value)
# rpc_cast_timeout=30

# Heartbeat frequency. (integer value)
# matchmaker_heartbeat_freq=300

# Heartbeat time-to-live. (integer value)
# matchmaker_heartbeat_ttl=600

# Size of RPC greenthread pool. (integer value)
# rpc_thread_pool_size=64

# Driver or drivers to handle sending notifications. (multi
# valued)
# notification_driver=
notification_driver=messagingv2

# AMQP topic used for OpenStack notifications. (list value)
# Deprecated group/name - [rpc_notifier2]/topics
# notification_topics=notifications

# Seconds to wait for a response from a call. (integer value)
# rpc_response_timeout=60
rpc_response_timeout=60

# A URL representing the messaging driver to use and its full
# configuration. If not set, we fall back to the rpc_backend
# option and driver specific configuration. (string value)
# transport_url=

# The messaging driver to use, defaults to rabbit. Other
# drivers include qpid and zmq. (string value)
# rpc_backend=rabbit
rpc_backend=rabbit

# The default exchange under which topics are scoped. May be
# overridden by an exchange name specified in the
# transport_url option. (string value)
# control_exchange=openstack
control_exchange=neutron
lock_path=/var/lib/neutron/lock


[matchmaker_redis]

#
# Options defined in oslo.messaging
#

# Host to locate redis. (string value)
# host=127.0.0.1

# Use this port to connect to redis host. (integer value)
# port=6379

# Password for Redis server (optional). (string value)
# password=


[matchmaker_ring]

#
# Options defined in oslo.messaging
#

# Matchmaker ring file (JSON). (string value)
# Deprecated group/name - [DEFAULT]/matchmaker_ringfile
# ringfile=/etc/oslo/matchmaker_ring.json

[quotas]
# Default driver to use for quota checks
# quota_driver = neutron.db.quota.driver.DbQuotaDriver

# Resource name(s) that are supported in quota features
# This option is deprecated for removal in the M release, please refrain from using it
# quota_items = network,subnet,port

# Default number of resource allowed per tenant. A negative value means
# unlimited.
# default_quota = -1

# Number of networks allowed per tenant. A negative value means unlimited.
# quota_network = 10

# Number of subnets allowed per tenant. A negative value means unlimited.
# quota_subnet = 10

# Number of ports allowed per tenant. A negative value means unlimited.
# quota_port = 50

# Number of security groups allowed per tenant. A negative value means
# unlimited.
# quota_security_group = 10

# Number of security group rules allowed per tenant. A negative value means
# unlimited.
# quota_security_group_rule = 100

# Number of vips allowed per tenant. A negative value means unlimited.
# quota_vip = 10

# Number of pools allowed per tenant. A negative value means unlimited.
# quota_pool = 10

# Number of pool members allowed per tenant. A negative value means unlimited.
# The default is unlimited because a member is not a real resource consumer
# on Openstack. However, on back-end, a member is a resource consumer
# and that is the reason why quota is possible.
# quota_member = -1

# Number of health monitors allowed per tenant. A negative value means
# unlimited.
# The default is unlimited because a health monitor is not a real resource
# consumer on Openstack. However, on back-end, a member is a resource consumer
# and that is the reason why quota is possible.
# quota_health_monitor = -1

# Number of loadbalancers allowed per tenant. A negative value means unlimited.
# quota_loadbalancer = 10

# Number of listeners allowed per tenant. A negative value means unlimited.
# quota_listener = -1

# Number of v2 health monitors allowed per tenant. A negative value means
# unlimited. These health monitors exist under the lbaas v2 API
# quota_healthmonitor = -1

# Number of routers allowed per tenant. A negative value means unlimited.
# quota_router = 10

# Number of floating IPs allowed per tenant. A negative value means unlimited.
# quota_floatingip = 50

# Number of firewalls allowed per tenant. A negative value means unlimited.
# quota_firewall = 1

# Number of firewall policies allowed per tenant. A negative value means
# unlimited.
# quota_firewall_policy = 1

# Number of firewall rules allowed per tenant. A negative value means
# unlimited.
# quota_firewall_rule = 100

# Default number of RBAC entries allowed per tenant. A negative value means
# unlimited.
# quota_rbac_policy = 10

[agent]
# Use "sudo neutron-rootwrap /etc/neutron/rootwrap.conf" to use the real
# root filter facility.
# Change to "sudo" to skip the filtering and just run the command directly
# root_helper = sudo neutron-rootwrap /etc/neutron/rootwrap.conf
root_helper = sudo neutron-rootwrap /etc/neutron/rootwrap.conf

# Set to true to add comments to generated iptables rules that describe
# each rule's purpose. (System must support the iptables comments module.)
# comment_iptables_rules = True

# Root helper daemon application to use when possible.
# root_helper_daemon = sudo neutron-rootwrap-daemon /etc/neutron/rootwrap.conf

# Use the root helper when listing the namespaces on a system. This may not
# be required depending on the security configuration. If the root helper is
# not required, set this to False for a performance improvement.
# use_helper_for_ns_read = True

# The interval to check external processes for failure in seconds (0=disabled)
# check_child_processes_interval = 60

# Action to take when an external process spawned by an agent dies
# Values:
#   respawn - Respawns the external process
#   exit - Exits the agent
# check_child_processes_action = respawn

# =========== items for agent management extension =============
# seconds between nodes reporting state to server; should be less than
# agent_down_time, best if it is half or less than agent_down_time
# report_interval = 30
report_interval = 30

# ===========  end of items for agent management extension =====

[keystone_authtoken]
auth_uri =
identity_uri =
admin_tenant_name = services
admin_user = neutron
admin_password =
project_name=services
password=
username=neutron
auth_plugin=password
user_domain_id=default
auth_url=
project_domain_id=default

[database]
# This line MUST be changed to actually run the plugin.
# Example:
# connection = mysql+pymysql://root:pass@127.0.0.1:3306/neutron
connection =
# Replace 127.0.0.1 above with the IP address of the database used by the
# main neutron server. (Leave it as is if the database runs on this host.)
# connection = sqlite://
# NOTE: In deployment the [database] section and its connection attribute may
# be set in the corresponding core plugin '.ini' file. However, it is suggested
# to put the [database] section and its connection attribute in this
# configuration file.

# Database engine for which script will be generated when using offline
# migration
# engine =

# The SQLAlchemy connection string used to connect to the slave database
# slave_connection =

# Database reconnection retry times - in event connectivity is lost
# set to -1 implies an infinite retry count
# max_retries = 10
max_retries = 10

# Database reconnection interval in seconds - if the initial connection to the
# database fails
# retry_interval = 10
retry_interval = 10

# Minimum number of SQL connections to keep open in a pool
# min_pool_size = 1
min_pool_size = 1

# Maximum number of SQL connections to keep open in a pool
# max_pool_size = 10
max_pool_size = 10

# Timeout in seconds before idle sql connections are reaped
# idle_timeout = 3600
idle_timeout = 3600

# If set, use this value for max_overflow with sqlalchemy
# max_overflow = 20
max_overflow = 20

# Verbosity of SQL debugging information. 0=None, 100=Everything
# connection_debug = 0

# Add python stack traces to SQL as comment strings
# connection_trace = False

# If set, use this value for pool_timeout with sqlalchemy
# pool_timeout = 10

[nova]
# Name of the plugin to load
# auth_plugin =
auth_plugin =password

# Config Section from which to load plugin specific options
# auth_section =

# PEM encoded Certificate Authority to use when verifying HTTPs connections.
# cafile =

# PEM encoded client certificate cert file
# certfile =

# Verify HTTPS connections.
# insecure = False

# PEM encoded client certificate key file
# keyfile =

# Name of nova region to use. Useful if keystone manages more than one region.
# region_name =
region_name =RegionOne

# Timeout value for http requests
# timeout =
project_name=services
username=nova
password=
project_domain_id=default
tenant_name=services
user_domain_id=default
auth_url=

[oslo_concurrency]

# Directory to use for lock files. For security, the specified directory should
# only be writable by the user running the processes that need locking.
# Defaults to environment variable OSLO_LOCK_PATH. If external locks are used,
# a lock path must be set.
# lock_path = $state_path/lock
lock_path = /var/lib/neutron/tmp

# Enables or disables inter-process locks.
# disable_process_locking = False

[oslo_policy]

# The JSON file that defines policies.
# policy_file = policy.json

# Default rule. Enforced when a requested rule is not found.
# policy_default_rule = default

# Directories where policy configuration files are stored.
# They can be relative to any directory in the search path defined by the
# config_dir option, or absolute paths. The file defined by policy_file
# must exist for these directories to be searched. Missing or empty
# directories are ignored.
# policy_dirs = policy.d

[oslo_messaging_amqp]

#
# From oslo.messaging
#

# Address prefix used when sending to a specific server (string value)
# Deprecated group/name - [amqp1]/server_request_prefix
# server_request_prefix = exclusive

# Address prefix used when broadcasting to all servers (string value)
# Deprecated group/name - [amqp1]/broadcast_prefix
# broadcast_prefix = broadcast

# Address prefix when sending to any server in group (string value)
# Deprecated group/name - [amqp1]/group_request_prefix
# group_request_prefix = unicast

# Name for the AMQP container (string value)
# Deprecated group/name - [amqp1]/container_name
# container_name =

# Timeout for inactive connections (in seconds) (integer value)
# Deprecated group/name - [amqp1]/idle_timeout
# idle_timeout = 0

# Debug: dump AMQP frames to stdout (boolean value)
# Deprecated group/name - [amqp1]/trace
# trace = false

# CA certificate PEM file for verifing server certificate (string value)
# Deprecated group/name - [amqp1]/ssl_ca_file
# ssl_ca_file =

# Identifying certificate PEM file to present to clients (string value)
# Deprecated group/name - [amqp1]/ssl_cert_file
# ssl_cert_file =

# Private key PEM file used to sign cert_file certificate (string value)
# Deprecated group/name - [amqp1]/ssl_key_file
# ssl_key_file =

# Password for decrypting ssl_key_file (if encrypted) (string value)
# Deprecated group/name - [amqp1]/ssl_key_password
# ssl_key_password =

# Accept clients using either SSL or plain TCP (boolean value)
# Deprecated group/name - [amqp1]/allow_insecure_clients
# allow_insecure_clients = false


[oslo_messaging_qpid]

#
# From oslo.messaging
#

# Use durable queues in AMQP. (boolean value)
# Deprecated group/name - [DEFAULT]/rabbit_durable_queues
# amqp_durable_queues = false

# Auto-delete queues in AMQP. (boolean value)
# Deprecated group/name - [DEFAULT]/amqp_auto_delete
# amqp_auto_delete = false

# Size of RPC connection pool. (integer value)
# Deprecated group/name - [DEFAULT]/rpc_conn_pool_size
# rpc_conn_pool_size = 30

# Qpid broker hostname. (string value)
# Deprecated group/name - [DEFAULT]/qpid_hostname
# qpid_hostname = localhost

# Qpid broker port. (integer value)
# Deprecated group/name - [DEFAULT]/qpid_port
# qpid_port = 5672

# Qpid HA cluster host:port pairs. (list value)
# Deprecated group/name - [DEFAULT]/qpid_hosts
# qpid_hosts = $qpid_hostname:$qpid_port

# Username for Qpid connection. (string value)
# Deprecated group/name - [DEFAULT]/qpid_username
# qpid_username =

# Password for Qpid connection. (string value)
# Deprecated group/name - [DEFAULT]/qpid_password
# qpid_password =

# Space separated list of SASL mechanisms to use for auth. (string value)
# Deprecated group/name - [DEFAULT]/qpid_sasl_mechanisms
# qpid_sasl_mechanisms =

# Seconds between connection keepalive heartbeats. (integer value)
# Deprecated group/name - [DEFAULT]/qpid_heartbeat
# qpid_heartbeat = 60

# Transport to use, either 'tcp' or 'ssl'. (string value)
# Deprecated group/name - [DEFAULT]/qpid_protocol
# qpid_protocol = tcp

# Whether to disable the Nagle algorithm. (boolean value)
# Deprecated group/name - [DEFAULT]/qpid_tcp_nodelay
# qpid_tcp_nodelay = true

# The number of prefetched messages held by receiver. (integer value)
# Deprecated group/name - [DEFAULT]/qpid_receiver_capacity
# qpid_receiver_capacity = 1

# The qpid topology version to use.  Version 1 is what was originally used by
# impl_qpid.  Version 2 includes some backwards-incompatible changes that allow
# broker federation to work.  Users should update to version 2 when they are
# able to take everything down, as it requires a clean break. (integer value)
# Deprecated group/name - [DEFAULT]/qpid_topology_version
# qpid_topology_version = 1


[oslo_messaging_rabbit]

#
# From oslo.messaging
#

# Use durable queues in AMQP. (boolean value)
# Deprecated group/name - [DEFAULT]/rabbit_durable_queues
# amqp_durable_queues = false
amqp_durable_queues = True

# Auto-delete queues in AMQP. (boolean value)
# Deprecated group/name - [DEFAULT]/amqp_auto_delete
# amqp_auto_delete = false

# Size of RPC connection pool. (integer value)
# Deprecated group/name - [DEFAULT]/rpc_conn_pool_size
# rpc_conn_pool_size = 30

# SSL version to use (valid only if SSL enabled). Valid values are TLSv1 and
# SSLv23. SSLv2, SSLv3, TLSv1_1, and TLSv1_2 may be available on some
# distributions. (string value)
# Deprecated group/name - [DEFAULT]/kombu_ssl_version
# kombu_ssl_version =

# SSL key file (valid only if SSL enabled). (string value)
# Deprecated group/name - [DEFAULT]/kombu_ssl_keyfile
# kombu_ssl_keyfile =

# SSL cert file (valid only if SSL enabled). (string value)
# Deprecated group/name - [DEFAULT]/kombu_ssl_certfile
# kombu_ssl_certfile =

# SSL certification authority file (valid only if SSL enabled). (string value)
# Deprecated group/name - [DEFAULT]/kombu_ssl_ca_certs
# kombu_ssl_ca_certs =

# How long to wait before reconnecting in response to an AMQP consumer cancel
# notification. (floating point value)
# Deprecated group/name - [DEFAULT]/kombu_reconnect_delay
# kombu_reconnect_delay = 1.0
kombu_reconnect_delay = 1.0

# The RabbitMQ broker address where a single node is used. (string value)
# Deprecated group/name - [DEFAULT]/rabbit_host
# rabbit_host = localhost

# The RabbitMQ broker port where a single node is used. (integer value)
# Deprecated group/name - [DEFAULT]/rabbit_port
# rabbit_port = 5672

# RabbitMQ HA cluster host:port pairs. (list value)
# Deprecated group/name - [DEFAULT]/rabbit_hosts
# rabbit_hosts = $rabbit_host:$rabbit_port
rabbit_hosts =

# Connect over SSL for RabbitMQ. (boolean value)
# Deprecated group/name - [DEFAULT]/rabbit_use_ssl
# rabbit_use_ssl = false
rabbit_use_ssl = False

# The RabbitMQ userid. (string value)
# Deprecated group/name - [DEFAULT]/rabbit_userid
# rabbit_userid = guest
rabbit_userid = openstack

# The RabbitMQ password. (string value)
# Deprecated group/name - [DEFAULT]/rabbit_password
# rabbit_password = guest
rabbit_password =

# The RabbitMQ login method. (string value)
# Deprecated group/name - [DEFAULT]/rabbit_login_method
# rabbit_login_method = AMQPLAIN

# The RabbitMQ virtual host. (string value)
# Deprecated group/name - [DEFAULT]/rabbit_virtual_host
# rabbit_virtual_host = /
rabbit_virtual_host = /

# How frequently to retry connecting with RabbitMQ. (integer value)
# rabbit_retry_interval = 1

# How long to backoff for between retries when connecting to RabbitMQ. (integer
# value)
# Deprecated group/name - [DEFAULT]/rabbit_retry_backoff
# rabbit_retry_backoff = 2

# Maximum number of RabbitMQ connection retries. Default is 0 (infinite retry
# count). (integer value)
# Deprecated group/name - [DEFAULT]/rabbit_max_retries
# rabbit_max_retries = 0

# Use HA queues in RabbitMQ (x-ha-policy: all). If you change this option, you
# must wipe the RabbitMQ database. (boolean value)
# Deprecated group/name - [DEFAULT]/rabbit_ha_queues
# rabbit_ha_queues = false
rabbit_ha_queues = True

# Deprecated, use rpc_backend=kombu+memory or rpc_backend=fake (boolean value)
# Deprecated group/name - [DEFAULT]/fake_rabbit
# fake_rabbit = false
heartbeat_rate=2
heartbeat_timeout_threshold=30

[qos]
# Drivers list to use to send the update notification
# notification_drivers = message_queue
notification_drivers = message_queue
```

### dhcp_agent.ini

```
[DEFAULT]
# Show debugging output in log (sets DEBUG log level output)
# debug = False
debug = False

# The DHCP agent will resync its state with Neutron to recover from any
# transient notification or rpc errors. The interval is number of
# seconds between attempts.
# resync_interval = 5
resync_interval = 30

# The DHCP agent requires an interface driver be set. Choose the one that best
# matches your plugin.
# interface_driver =
interface_driver =neutron.agent.linux.interface.OVSInterfaceDriver

# Example of interface_driver option for OVS based plugins(OVS, Ryu, NEC, NVP,
# BigSwitch/Floodlight)
# interface_driver = neutron.agent.linux.interface.OVSInterfaceDriver

# Name of Open vSwitch bridge to use
# ovs_integration_bridge = br-int

# Use veth for an OVS interface or not.
# Support kernels with limited namespace support
# (e.g. RHEL 6.5) so long as ovs_use_veth is set to True.
# ovs_use_veth = False

# Example of interface_driver option for LinuxBridge
# interface_driver = neutron.agent.linux.interface.BridgeInterfaceDriver

# MTU setting for device.
# network_device_mtu =

# The agent can use other DHCP drivers.  Dnsmasq is the simplest and requires
# no additional setup of the DHCP server.
# dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq

# Allow overlapping IP (Must have kernel build with CONFIG_NET_NS=y and
# iproute2 package that supports namespaces). This option is deprecated and
# will be removed in a future release, at which point the old behavior of
# use_namespaces = True will be enforced.
# use_namespaces = True

# In some cases the neutron router is not present to provide the metadata
# IP but the DHCP server can be used to provide this info. Setting this
# value will force the DHCP server to append specific host routes to the
# DHCP request. If this option is set, then the metadata service will be
# activated for all the networks.
# force_metadata = False
force_metadata = False

# The DHCP server can assist with providing metadata support on isolated
# networks. Setting this value to True will cause the DHCP server to append
# specific host routes to the DHCP request. The metadata service will only
# be activated when the subnet does not contain any router port. The guest
# instance must be configured to request host routes via DHCP (Option 121).
# This option doesn't have any effect when force_metadata is set to True.
# enable_isolated_metadata = False
enable_isolated_metadata = True

# Allows for serving metadata requests coming from a dedicated metadata
# access network whose cidr is 169.254.169.254/16 (or larger prefix), and
# is connected to a Neutron router from which the VMs send metadata
# request. In this case DHCP Option 121 will not be injected in VMs, as
# they will be able to reach 169.254.169.254 through a router.
# This option requires enable_isolated_metadata = True
# enable_metadata_network = False
enable_metadata_network = False

# Number of threads to use during sync process. Should not exceed connection
# pool size configured on server.
# num_sync_threads = 4

# Location to store DHCP server config files
# dhcp_confs = $state_path/dhcp

# Domain to use for building the hostnames. This option will be deprecated in
# a future release. It is being replaced by dns_domain in neutron.conf
# dhcp_domain = openstacklocal
dhcp_domain = openstacklocal

# Override the default dnsmasq settings with this file
# dnsmasq_config_file =

# Comma-separated list of DNS servers which will be used by dnsmasq
# as forwarders.
# dnsmasq_dns_servers =

# Base log dir for dnsmasq logging. The log contains DHCP and DNS log
# information and is useful for debugging issues with either DHCP or DNS.
# If this section is null, disable dnsmasq log.
# dnsmasq_base_log_dir =

# Limit number of leases to prevent a denial-of-service.
# dnsmasq_lease_max = 16777216

# Location to DHCP lease relay UNIX domain socket
# dhcp_lease_relay_socket = $state_path/dhcp/lease_relay

# Use broadcast in DHCP replies
# dhcp_broadcast_reply = False
dhcp_broadcast_reply = False

# dhcp_delete_namespaces, which is True by default, can be set to False if
# namespaces can't be deleted cleanly on the host running the DHCP agent.
# Disable this if you hit the issue in
# https://bugs.launchpad.net/neutron/+bug/1052535 or if
# you are sure that your version of iproute suffers from the problem.
# This should not be a problem any more.  Refer to bug:
# https://bugs.launchpad.net/neutron/+bug/1418079
# This option is deprecated and will be removed in the M release
# dhcp_delete_namespaces = True
dhcp_delete_namespaces = True

# Timeout for ovs-vsctl commands.
# If the timeout expires, ovs commands will fail with ALARMCLOCK error.
# ovs_vsctl_timeout = 10
root_helper=sudo neutron-rootwrap /etc/neutron/rootwrap.conf
state_path=/var/lib/neutron

[AGENT]
# Log agent heartbeats from this DHCP agent
# log_agent_heartbeats = False
```

### dnsmasq-neutron.conf
dhcp-option-force=26,1400

### l3_agent.ini

```
[DEFAULT]
# Show debugging output in log (sets DEBUG log level output)
# debug = False
debug = False

# L3 requires that an interface driver be set. Choose the one that best
# matches your plugin.
# interface_driver =
interface_driver =neutron.agent.linux.interface.OVSInterfaceDriver

# Example of interface_driver option for OVS based plugins (OVS, Ryu, NEC)
# that supports L3 agent
# interface_driver = neutron.agent.linux.interface.OVSInterfaceDriver

# Use veth for an OVS interface or not.
# Support kernels with limited namespace support
# (e.g. RHEL 6.5) so long as ovs_use_veth is set to True.
# ovs_use_veth = False

# Example of interface_driver option for LinuxBridge
# interface_driver = neutron.agent.linux.interface.BridgeInterfaceDriver

# Allow overlapping IP (Must have kernel build with CONFIG_NET_NS=y and
# iproute2 package that supports namespaces). This option is deprecated and
# will be removed in a future release, at which point the old behavior of
# use_namespaces = True will be enforced.
# use_namespaces = True

# If use_namespaces is set as False then the agent can only configure one router.

# This is done by setting the specific router_id.
# router_id =

# When external_network_bridge is set, each L3 agent can be associated
# with no more than one external network. This value should be set to the UUID
# of that external network. To allow L3 agent support multiple external
# networks, both the external_network_bridge and gateway_external_network_id
# must be left empty.
gateway_external_network_id =

# With IPv6, the network used for the external gateway does not need
# to have an associated subnet, since the automatically assigned
# link-local address (LLA) can be used. However, an IPv6 gateway address
# is needed for use as the next-hop for the default route. If no IPv6
# gateway address is configured here, (and only then) the neutron router
# will be configured to get its default route from router advertisements (RAs)
# from the upstream router; in which case the upstream router must also be
# configured to send these RAs.
# The ipv6_gateway, when configured, should be the LLA of the interface
# on the upstream router. If a next-hop using a global unique address (GUA)
# is desired, it needs to be done via a subnet allocated to the network
# and not through this parameter.
# ipv6_gateway =

# (StrOpt) Driver used for ipv6 prefix delegation. This needs to be
# an entry point defined in the neutron.agent.linux.pd_drivers namespace. See
# setup.cfg for entry points included with the neutron source.
# prefix_delegation_driver = dibbler

# Indicates that this L3 agent should also handle routers that do not have
# an external network gateway configured.  This option should be True only
# for a single agent in a Neutron deployment, and may be False for all agents
# if all routers must have an external network gateway
# handle_internal_only_routers = True
handle_internal_only_routers = True

# Name of bridge used for external network traffic. This should be set to
# empty value for the linux bridge. when this parameter is set, each L3 agent
# can be associated with no more than one external network.
# This option is deprecated and will be removed in the M release.
# external_network_bridge = br-ex
xternal_network_bridge =
#external_network_bridge = br-ex

# TCP Port used by Neutron metadata server
# metadata_port = 9697
metadata_port = 9697

# Send this many gratuitous ARPs for HA setup. Set it below or equal to 0
# to disable this feature.
# send_arp_for_ha = 3
send_arp_for_ha = 3

# seconds between re-sync routers' data if needed
# periodic_interval = 40
periodic_interval = 40

# seconds to start to sync routers' data after
# starting agent
# periodic_fuzzy_delay = 5
periodic_fuzzy_delay = 5

# enable_metadata_proxy, which is true by default, can be set to False
# if the Nova metadata server is not available
# enable_metadata_proxy = True
enable_metadata_proxy = True

# Iptables mangle mark used to mark metadata valid requests
# metadata_access_mark = 0x1

# Iptables mangle mark used to mark ingress from external network
# external_ingress_mark = 0x2

# router_delete_namespaces, which is True by default, can be set to False if
# namespaces can't be deleted cleanly on the host running the L3 agent.
# Disable this if you hit the issue in
# https://bugs.launchpad.net/neutron/+bug/1052535 or if
# you are sure that your version of iproute suffers from the problem.
# If True, namespaces will be deleted when a router is destroyed.
# This should not be a problem any more.  Refer to bug:
# https://bugs.launchpad.net/neutron/+bug/1418079
# This option is deprecated and will be removed in the M release
# router_delete_namespaces = True
router_delete_namespaces = True

# Timeout for ovs-vsctl commands.
# If the timeout expires, ovs commands will fail with ALARMCLOCK error.
# ovs_vsctl_timeout = 10

# The working mode for the agent. Allowed values are:
# - legacy: this preserves the existing behavior where the L3 agent is
#   deployed on a centralized networking node to provide L3 services
#   like DNAT, and SNAT. Use this mode if you do not want to adopt DVR.
# - dvr: this mode enables DVR functionality, and must be used for an L3
#   agent that runs on a compute host.
# - dvr_snat: this enables centralized SNAT support in conjunction with
#   DVR. This mode must be used for an L3 agent running on a centralized
#   node (or in single-host deployments, e.g. devstack).
# agent_mode = legacy
agent_mode = dvr_snat

# Location to store keepalived and all HA configurations
# ha_confs_path = $state_path/ha_confs

# VRRP authentication type AH/PASS
# ha_vrrp_auth_type = PASS

# VRRP authentication password
# ha_vrrp_auth_password =

# The advertisement interval in seconds
# ha_vrrp_advert_int = 2

[AGENT]
# Log agent heartbeats from this L3 agent
# log_agent_heartbeats = False
```

### lbaas_agent.ini

```
[DEFAULT]
# Show debugging output in log (sets DEBUG log level output).
# debug = False
debug = False

# The LBaaS agent will resync its state with Neutron to recover from any
# transient notification or rpc errors. The interval is number of
# seconds between attempts.
# periodic_interval = 10

# LBaas requires an interface driver be set. Choose the one that best
# matches your plugin.
# interface_driver =
interface_driver =neutron.agent.linux.interface.OVSInterfaceDriver

# Example of interface_driver option for OVS based plugins (OVS, Ryu, NEC, NVP,
# BigSwitch/Floodlight)
# interface_driver = neutron.agent.linux.interface.OVSInterfaceDriver

# Use veth for an OVS interface or not.
# Support kernels with limited namespace support
# (e.g. RHEL 6.5) so long as ovs_use_veth is set to True.
# ovs_use_veth = False

# Example of interface_driver option for LinuxBridge
# interface_driver = neutron.agent.linux.interface.BridgeInterfaceDriver

# The agent requires drivers to manage the loadbalancer.  HAProxy is the opensource version.
# Multiple device drivers reflecting different service providers could be specified:
# device_driver = path.to.provider1.driver.Driver
device_driver = neutron_lbaas.drivers.haproxy.namespace_driver.HaproxyNSDriver
# device_driver = path.to.provider2.driver.Driver
# Default is:

[haproxy]
# Location to store config and state files
# loadbalancer_state_path = $state_path/lbaas

# The user group
# user_group = nogroup
user_group = haproxy

# When delete and re-add the same vip, send this many gratuitous ARPs to flush
# the ARP cache in the Router. Set it below or equal to 0 to disable this feature.
# send_gratuitous_arp = 3
```

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

### neutron_vpnaas.conf
```
[service_providers]
# Must be in form:
# service_provider=<service_type>:<name>:<driver>[:default]
# List of allowed service types includes VPN
# Combination of <service type> and <name> must be unique; <driver> must also be unique
# This is multiline option
#service_provider=VPN:openswan:neutron_vpnaas.services.vpn.service_drivers.ipsec.IPsecVPNDriver:default
service_provider=VPN:strongswan:neutron_vpnaas.services.vpn.service_drivers.ipsec.IPsecVPNDriver:default
# Uncomment the following line (and comment out the OpenSwan VPN line) to enable Cisco's VPN driver.
# service_provider=VPN:cisco:neutron_vpnaas.services.vpn.service_drivers.cisco_ipsec.CiscoCsrIPsecVPNDriver:default
# Uncomment the following line (and comment out the OpenSwan VPN line) to enable Brocade Vyatta's VPN driver.
# service_provider=VPN:vyatta:neutron_vpnaas.services.vpn.service_drivers.vyatta_ipsec.VyattaIPsecDriver:default
```

### ml2_conf.ini
```
[ml2]
# (ListOpt) List of network type driver entrypoints to be loaded from
# the neutron.ml2.type_drivers namespace.
#
# type_drivers = local,flat,vlan,gre,vxlan,geneve
type_drivers = vlan,vxlan,gre,local
# Example: type_drivers = flat,vlan,gre,vxlan,geneve

# (ListOpt) Ordered list of network_types to allocate as tenant
# networks. The default value 'local' is useful for single-box testing
# but provides no connectivity between hosts.
#
# tenant_network_types = local
tenant_network_types = vxlan
#,vlan
# Example: tenant_network_types = vlan,gre,vxlan,geneve


# (ListOpt) Ordered list of networking mechanism driver entrypoints
# to be loaded from the neutron.ml2.mechanism_drivers namespace.
# mechanism_drivers =
mechanism_drivers =openvswitch,l2population
# Example: mechanism_drivers = openvswitch,mlnx
# Example: mechanism_drivers = arista
# Example: mechanism_drivers = openvswitch,cisco_nexus,logger
# Example: mechanism_drivers = openvswitch,brocade
# Example: mechanism_drivers = linuxbridge,brocade

# (ListOpt) Ordered list of extension driver entrypoints
# to be loaded from the neutron.ml2.extension_drivers namespace.
#extension_drivers = dns-integration
# Example: extension_drivers = anewextensiondriver

# =========== items for MTU selection and advertisement =============
# (IntOpt) Path MTU.  The maximum permissible size of an unfragmented
# packet travelling from and to addresses where encapsulated Neutron
# traffic is sent.  Drivers calculate maximum viable MTU for
# validating tenant requests based on this value (typically,
# path_mtu - max encap header size).  If <=0, the path MTU is
# indeterminate and no calculation takes place.
# path_mtu = 0
path_mtu = 0

# (IntOpt) Segment MTU.  The maximum permissible size of an
# unfragmented packet travelling a L2 network segment.  If <=0,
# the segment MTU is indeterminate and no calculation takes place.
# segment_mtu = 0

# (ListOpt) Physical network MTUs.  List of mappings of physical
# network to MTU value.  The format of the mapping is
# <physnet>:<mtu val>.  This mapping allows specifying a
# physical network MTU value that differs from the default
# segment_mtu value.
# physical_network_mtus =
# Example: physical_network_mtus = physnet1:1550, physnet2:1500
# ======== end of items for MTU selection and advertisement =========

# (StrOpt) Default network type for external networks when no provider
# attributes are specified. By default it is None, which means that if
# provider attributes are not specified while creating external networks
# then they will have the same type as tenant networks.
# Allowed values for external_network_type config option depend on the
# network type values configured in type_drivers config option.
# external_network_type =
# Example: external_network_type = local

[ml2_type_flat]
# (ListOpt) List of physical_network names with which flat networks
# can be created. Use * to allow flat networks with arbitrary
# physical_network names.
#
# flat_networks =
flat_networks =physnet1
# Example:flat_networks = physnet1,physnet2
# Example:flat_networks = *

[ml2_type_vlan]
# (ListOpt) List of <physical_network>[:<vlan_min>:<vlan_max>] tuples
# specifying physical_network names usable for VLAN provider and
# tenant networks, as well as ranges of VLAN tags on each
# physical_network available for allocation as tenant networks.
#
# network_vlan_ranges =
network_vlan_ranges =physnet3:5:60
#network_vlan_ranges =physnet3:1:10,physnet2
# Example: network_vlan_ranges = physnet1:1000:2999,physnet2

[ml2_type_gre]
# (ListOpt) Comma-separated list of <tun_min>:<tun_max> tuples enumerating ranges of GRE tunnel IDs that are available for tenant network allocation
# tunnel_id_ranges =
tunnel_id_ranges =1:20

[ml2_type_vxlan]
# (ListOpt) Comma-separated list of <vni_min>:<vni_max> tuples enumerating
# ranges of VXLAN VNI IDs that are available for tenant network allocation.
#
# vni_ranges =
vni_ranges =20:40

# (StrOpt) Multicast group for the VXLAN interface. When configured, will
# enable sending all broadcast traffic to this multicast group. When left
# unconfigured, will disable multicast VXLAN mode.
#
# vxlan_group =
vxlan_group =239.1.1.1
# Example: vxlan_group = 239.1.1.1

[ml2_type_geneve]
# (ListOpt) Comma-separated list of <vni_min>:<vni_max> tuples enumerating
# ranges of Geneve VNI IDs that are available for tenant network allocation.
#
# vni_ranges =

# (IntOpt) Geneve encapsulation header size is dynamic, this
# value is used to calculate the maximum MTU for the driver.
# this is the sum of the sizes of the outer ETH+IP+UDP+GENEVE
# header sizes.
# The default size for this field is 50, which is the size of the
# Geneve header without any additional option headers
#
# max_header_size =
# Example: max_header_size = 50 (Geneve headers with no additional options)

[securitygroup]
# Controls if neutron security group is enabled or not.
# It should be false when you use nova security group.
# enable_security_group = True
enable_security_group = True

# Use ipset to speed-up the iptables security groups. Enabling ipset support
# requires that ipset is installed on L2 agent node.
enable_ipset = True
```

### openvswitch_agent.ini
```
[ovs]
# Do not change this parameter unless you have a good reason to.
# This is the name of the OVS integration bridge. There is one per hypervisor.
# The integration bridge acts as a virtual "patch bay". All VM VIFs are
# attached to this bridge and then "patched" according to their network
# connectivity.
#
# integration_bridge = br-int
integration_bridge = br-int

# Only used for the agent if tunnel_id_ranges is not empty for
# the server.  In most cases, the default value should be fine.
#
# tunnel_bridge = br-tun
tunnel_bridge = br-tun

# Peer patch port in integration bridge for tunnel bridge
# int_peer_patch_port = patch-tun

# Peer patch port in tunnel bridge for integration bridge
# tun_peer_patch_port = patch-int

# Uncomment this line for the agent if tunnel_id_ranges is not
# empty for the server. Set local-ip to be the local IP address of
# this hypervisor.
#
# local_ip =
local_ip = 192.168.100.233

# (ListOpt) Comma-separated list of <physical_network>:<bridge> tuples
# mapping physical network names to the agent's node-specific OVS
# bridge names to be used for flat and VLAN networks. The length of
# bridge names should be no more than 11. Each bridge must
# exist, and should have a physical network interface configured as a
# port. All physical networks configured on the server should have
# mappings to appropriate bridges on each agent.
#
# Note: If you remove a bridge from this mapping, make sure to disconnect it
# from the integration bridge as it won't be managed by the agent anymore.
#
# bridge_mappings =
bridge_mappings = physnet3:ovsbr3
#,physnet2:ovsbr2
# Example: bridge_mappings = physnet1:br-eth1

# (BoolOpt) Use veths instead of patch ports to interconnect the integration
# bridge to physical networks. Support kernel without ovs patch port support
# so long as it is set to True.
# use_veth_interconnection = False

# (StrOpt) Which OVSDB backend to use, defaults to 'vsctl'
# vsctl - The backend based on executing ovs-vsctl
# native - The backend based on using native OVSDB
ovsdb_interface = native

# (StrOpt) The connection string for the native OVSDB backend
# To enable ovsdb-server to listen on port 6640:#   ovs-vsctl set-manager ptcp:6640:127.0.0.1
ovsdb_connection = tcp:127.0.0.1:6640

# (StrOpt) OpenFlow interface to use.
# 'ovs-ofctl' or 'native'.
# of_interface = ovs-ofctl
#
# (IPOpt)
# Address to listen on for OpenFlow connections.
# Used only for 'native' driver.
# of_listen_address = 127.0.0.1
#
# (IntOpt)
# Port to listen on for OpenFlow connections.
# Used only for 'native' driver.
# of_listen_port = 6633
#
# (IntOpt)
# Timeout in seconds to wait for the local switch connecting the controller.
# Used only for 'native' driver.
# of_connect_timeout=30
#
# (IntOpt)
# Timeout in seconds to wait for a single OpenFlow request.
# Used only for 'native' driver.
# of_request_timeout=10

# (StrOpt) ovs datapath to use.
# 'system' is the default value and corresponds to the kernel datapath.
# To enable the userspace datapath set this value to 'netdev'
# datapath_type = system
enable_tunneling=True

[agent]
# Log agent heartbeats from this OVS agent
# log_agent_heartbeats = False

# Agent's polling interval in seconds
# polling_interval = 2
polling_interval = 2

# Minimize polling by monitoring ovsdb for interface changes
# minimize_polling = True

# When minimize_polling = True, the number of seconds to wait before
# respawning the ovsdb monitor after losing communication with it
# ovsdb_monitor_respawn_interval = 30

# (ListOpt) The types of tenant network tunnels supported by the agent.
# Setting this will enable tunneling support in the agent. This can be set to
# either 'gre' or 'vxlan'. If this is unset, it will default to [] and
# disable tunneling support in the agent.
# You can specify as many values here as your compute hosts supports.
#
# tunnel_types =
tunnel_types =vxlan
# Example: tunnel_types = gre
# Example: tunnel_types = vxlan
# Example: tunnel_types = vxlan, gre

# (IntOpt) The port number to utilize if tunnel_types includes 'vxlan'. By
# default, this will make use of the Open vSwitch default value of '4789' if
# not specified.
#
# vxlan_udp_port =
vxlan_udp_port =4789
# Example: vxlan_udp_port = 8472

# (IntOpt) This is the MTU size of veth interfaces.
# Do not change unless you have a good reason to.
# The default MTU size of veth interfaces is 1500.
# This option has no effect if use_veth_interconnection is False
# veth_mtu =
# Example: veth_mtu = 1504

# (BoolOpt) Flag to enable l2-population extension. This option should only be
# used in conjunction with ml2 plugin and l2population mechanism driver. It'll
# enable plugin to populate remote ports macs and IPs (using fdb_add/remove
# RPC calbbacks instead of tunnel_sync/update) on OVS agents in order to
# optimize tunnel management.
#
l2_population = True
#l2_population = True

# Enable local ARP responder. Requires OVS 2.1. This is only used by the l2
# population ML2 MechanismDriver.
#
# arp_responder = False
arp_responder = True

# Enable suppression of ARP responses that don't match an IP address that
# belongs to the port from which they originate.
# Note: This prevents the VMs attached to this agent from spoofing,
# it doesn't protect them from other devices which have the capability to spoof
# (e.g. bare metal or VMs attached to agents without this flag set to True).
# Requires a version of OVS that can match ARP headers.
#
# prevent_arp_spoofing = True
prevent_arp_spoofing = True

# (BoolOpt) Set or un-set the don't fragment (DF) bit on outgoing IP packet
# carrying GRE/VXLAN tunnel. The default value is True.
#
# dont_fragment = True

# (BoolOpt) Set to True on L2 agents to enable support
# for distributed virtual routing.
#
#enable_distributed_routing = False
enable_distributed_routing = True

# (IntOpt) Set new timeout in seconds for new rpc calls after agent receives
# SIGTERM. If value is set to 0, rpc timeout won't be changed"
#
# quitting_rpc_timeout = 10

# (ListOpt) Extensions list to use
# Example: extensions = qos
#
# extensions =

# (BoolOpt) Set or un-set the checksum on outgoing IP packet
# carrying GRE/VXLAN tunnel. The default value is False.
#
# tunnel_csum = False

# (StrOpt) agent_type to report.
# This config entry allows configuration of the neutron agent type reported
# by the default ovs l2 agent. This allows multiple ovs mechanism drivers
# to share a common ovs agent implementation. NOTE: this value will be
# removed in the mitaka cycle.
#
# agent_type = 'Open vSwitch agent'
drop_flows_on_start=False

[securitygroup]
# Firewall driver for realizing neutron security group function.
# firewall_driver = neutron.agent.firewall.NoopFirewallDriver
firewall_driver = neutron.agent.linux.iptables_firewall.OVSHybridIptablesFirewallDriver
# Example: firewall_driver = neutron.agent.linux.iptables_firewall.OVSHybridIptablesFirewallDriver

# Controls if neutron security group is enabled or not.
# It should be false when you use nova security group.
# enable_security_group = True

#-----------------------------------------------------------------------------
# Sample Configurations.
#-----------------------------------------------------------------------------
#
# 1. With VLANs on eth1.
# [ovs]
# integration_bridge = br-int
# bridge_mappings = default:br-eth1
#
# 2. With GRE tunneling.
# [ovs]
# integration_bridge = br-int
# tunnel_bridge = br-tun
# local_ip = 10.0.0.3
#
# 3. With VXLAN tunneling.
# [ovs]
# integration_bridge = br-int
# tunnel_bridge = br-tun
# local_ip = 10.0.0.3
# [agent]
# tunnel_types = vxlan
```

### rootwrap.conf
```
# Configuration for neutron-rootwrap
# This file should be owned by (and only-writeable by) the root user

[DEFAULT]
# List of directories to load filter definitions from (separated by ',').
# These directories MUST all be only writeable by root !
filters_path=/etc/neutron/rootwrap.d,/usr/share/neutron/rootwrap

# List of directories to search executables in, in case filters do not
# explicitely specify a full path (separated by ',')
# If not specified, defaults to system PATH environment variable.
# These directories MUST all be only writeable by root !
exec_dirs=/sbin,/usr/sbin,/bin,/usr/bin,/usr/local/bin,/usr/local/sbin

# Enable logging to syslog
# Default value is False
use_syslog=False

# Which syslog facility to use.
# Valid values include auth, authpriv, syslog, local0, local1...
# Default value is 'syslog'
syslog_log_facility=syslog

# Which messages to log.
# INFO means log all usage
# ERROR means only log unsuccessful attempts
syslog_log_level=ERROR

[xenapi]
# XenAPI configuration is only required by the L2 agent if it is to
# target a XenServer/XCP compute host's dom0.
xenapi_connection_url=<None>
xenapi_connection_username=root
xenapi_connection_password=<None>
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

### vpn_agent.ini
```
[DEFAULT]
# VPN-Agent configuration file
# Note vpn-agent inherits l3-agent, so you can use configs on l3-agent also

[vpnagent]
# vpn device drivers which vpn agent will use
# If we want to use multiple drivers,  we need to define this option multiple times.
# NOTE: StrongSwan and openSwan cannot be installed at the same time. Thus, both cannot
#       be enabled for use. In the future when flavors/STF support is available,
#       this will still constrain the flavors which can be used together.
# vpn_device_driver=neutron_vpnaas.services.vpn.device_drivers.ipsec.OpenSwanDriver
# vpn_device_driver=neutron_vpnaas.services.vpn.device_drivers.cisco_ipsec.CiscoCsrIPsecDriver
# vpn_device_driver=neutron_vpnaas.services.vpn.device_drivers.vyatta_ipsec.VyattaIPSecDriver
# vpn_device_driver=neutron_vpnaas.services.vpn.device_drivers.strongswan_ipsec.StrongSwanDriver
vpn_device_driver=neutron_vpnaas.services.vpn.device_drivers.fedora_strongswan_ipsec.FedoraStrongSwanDriver
# vpn_device_driver=neutron_vpnaas.services.vpn.device_drivers.libreswan_ipsec.LibreSwanDriver
# vpn_device_driver=another_driver

[ipsec]
# Status check interval
# ipsec_status_check_interval=60

# Enable detail logging for ipsec pluto process.
# If the flag set to True, the detailed logging will
# be written into config_base_dir/<pid>/logs."
# NOTE: this applies to OpenSwan and Libraswan, and
# that StrongSwan has logging that logs to syslog.
# enable_detailed_logging=False

[strongswan]
# For fedora use:
default_config_area=/usr/share/strongswan/templates/config/strongswan.d
# Default is for ubuntu use, /etc/strongswan.d
# default_config_area=/etc/strongswan.d

[libreswan]
# Initial interval in seconds for checking if pluto daemon is shutdown
# shutdown_check_timeout=1
#
# The maximum number of retries for checking for pluto daemon shutdown
# shutdown_check_retries=5
#
# A factor to increase the retry interval for each retry
# shutdown_check_back_off=1.5
```