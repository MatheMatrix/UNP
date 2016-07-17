### Neutron DHCP Agent

#### 启动进程
```
/usr/bin/python2 /usr/bin/neutron-dhcp-agent \
--config-file /usr/share/neutron/neutron-dist.conf \
--config-file /etc/neutron/neutron.conf \
--config-file /etc/neutron/dhcp_agent.ini \
--config-dir /etc/neutron/conf.d/common \
--config-dir /etc/neutron/conf.d/neutron-dhcp-agent \
--log-file /var/log/neutron/dhcp-agent.log
```

#### 需要注意的参数
1. `dnsmasq_dns_servers`。递归 DNS 服务器，即当 dnsmasq 进程无法完成 DNS 解析时，会 forward 到该地址上进行 DNS 解析。
    建议使用运营商给出的 DNS 地址，或者统一配置成 114DNS 或 DNSPOD 等厂商提供的 DNS：114.114.114.114, 119.29.29.29。
2. `dnsmasq_base_log_dir`。配置 DHCP 和 DNS 解析时产生日志的存放路径。一般不配置此参数，
    如客户有需要，将此选项配置成存放日志的路径，例如配置成 `/tmp/dnsmasq/`。并且建议配置相关 logrotate，
    添加 logrotate 配置文件 `/etc/logrotate.d/openstack-neutron-dnsmasq` ，内容如下：
    ```
    /tmp/dnsmasq/* {
        rotate 14
        size 10M
        missingok
        compress
        copytruncate
    }
    ```
3. `interface_driver`。一般不需要修改，配置成 neutron.agent.linux.interface.OVSInterfaceDriver
4. `dhcp_driver`。 一般不需要修改，配置成 neutron.agent.linux.dhcp.Dnsmasq
5. `log_agent_heartbeats`。设置成 True。该选项会在 Agent 进行上报状态时进行日志的打印。

#### dhcp.conf

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
```
dhcp-option-force=26,1400
```
