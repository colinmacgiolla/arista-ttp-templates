# Arista Template Text Parser Templates

[Arista Networks](https://www.arista.com) has been doing a great deal of work on the [Arista Validated Design](https://github.com/aristanetworks/ansible-avd) which aims to generate configuration from structured data through a set of ansible playbooks and jinja2 templates.
[Template Text Parser](https://pypi.org/project/ttp/) is basically the inverse - it takes a templates and attempts to generate structured data from the provided inputs

This is the beginning of a set of templates, using TTP, and the AVD data structures to try and generate compatible strucuted outputs that could later be used (if you wanted) to feed in through the AVD config generation.


# Example

So far I've just created a single template for [VLAN interfaces/SVIs](./templates/vlan-interfaces-ttp.j2) but as an example:

## Sample Configuration
Obviously not a correct config, but really just to demonstrate the population of state
```
interface Vlan246
   ip proxy-arp
   ip address 10.10.20.2/23
   vrrp 48 priority 105
   vrrp 48 preempt delay minimum 180
   vrrp 48 preempt delay reload 180
   vrrp 48 authentication text security123
   vrrp 48 ip 100.10.20.1
   shutdown
   vrf Test2
   ip ospf network point-to-point
   ip ospf authentication message-digest
   pim ipv4 sparse-mode
!
```

And the json output:
```json
[
	[
		{
			"interfaces": {
				"svi": {
					"id": "246",
					"ip": "10.10.20.2",
					"netmask": "23",
					"ospf_authentication": "message-digest",
					"ospf_network_point_to_point": true,
					"pim": {
						"ipv4": {
							"sparse_mode": true
						}
					},
					"shutdown": true,
					"vrf": "Test2",
					"vrrp": {
						"48": {
							"authText": "security123",
							"groupPriority": "105",
							"ipv4": "100.10.20.1",
							"preemptMin": "180",
							"preemptReload": "180"
						}
					}
				}
			}
		}
	]
]
```