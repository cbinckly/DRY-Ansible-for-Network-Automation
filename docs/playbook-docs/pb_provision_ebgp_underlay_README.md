
# Provision EBGP Underlay 

Playbook name: _pb_provision_ebgp_underlay.yml_

This playbook first discovers the network topology, then generates the Junos configuration for underlay IP connectivity 
and EBGP peering over the physical interfaces to redistribute the loopback addresses.


## Example 1 - Basic

1. Create a group called `ip_underlay` in your inventory file whose members are the devices you want to be part of 
the EBGP underaly:

    ```
    # invenotry/hosts.ini 
    
    [ip_underlay]
    router-1
    router-2
    router-3
    ```
2. Run the playbook:

    ```
    ansible-playbook pb_provision_ebgp_underlay.yml -i invenotry/hosts.ini
    ```
    

Result: Generate IP and EBGP configuration files and store them in the following folder:

```
inventory
├── _ebgp_underlay_configs
│   ├── ebgp_underlay.router-1.conf
│   ├── ebgp_underlay.router-2.conf
│   ├── ebgp_underlay.router-3.conf
│   ├── ip_underlay.router-1.conf
│   ├── ip_underlay.router-2.conf
│   └── ip_underlay.router-3.conf
```
Example of output configuration 

```
# ip_underlay.router-1.conf

interfaces {
    xe-0/0/1 {
        mtu 9216;
        unit 0 {
            family inet {
                address 10.100.0.0/31;
            }
        }
    }
    xe-0/0/2 {
        mtu 9216;
        unit 0 {
            family inet {
                address 10.100.0.2/31;
            }
        }
    }
}
```

```
# ebpg_underlay.router-1.conf

protocols {
    bgp {
        group ebgp-underlay {
                type external;
                family inet {
                unicast;
                }
                multipath {
                    multiple-as;
                }
                export pl-local_loopback;
                local-as 4200000101;
    
                neighbor 10.100.0.1 {
                    description router-2;
                    peer-as 4200000102;
                }
                neighbor 10.100.0.3 {
                    description router-3;
                    peer-as 4200000103;
                }
            }
        }
}

policy-options {
    policy-statement pl-local_loopback {
        term 1 {
            from {
                protocol direct;
                interface lo0.0;
            }
            then accept;
        }
    }

policy-statement ECMP {
        then {
            load-balance per-packet;
        }
    }
}

routing-options {
    forwarding-table {
        export ECMP;
    }
}
```

## Example 3 - Specify AS Numbers

The Autonomous System Numbers (ASNs) are generated incrementally starting from a default value `4200000100` 
(32-bits format). 
You can change the seed value by modifying the variable `asn_start`. 
Example:
```yaml
# invenotry/group_vars/all.yml

asn_start: 65001
```


## Complete List of Variables

* `targets` (default value = `all`): it is the _hosts_ parameter of the playbook, used to set the target hosts. 
It can be a group name or a device name

