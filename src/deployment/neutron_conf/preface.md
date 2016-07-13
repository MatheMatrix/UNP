### 概述

Neutron 配置主要涉及 Neutron Server 以及各个 agent 配置文件的变更，
关于 Neutron 所有配置文件的目录树如下：

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

### 需要注意的参数

下面列举一些 Neutron 各个 agent 都要配置的，并且配置相同的选项：

1. `interface_driver` 配置成 neutron.agent.linux.interface.OVSInterfaceDriver
2. `log_agent_heartbeats` 配置成 True
