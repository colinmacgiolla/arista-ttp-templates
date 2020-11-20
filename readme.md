# (unofficial) Arista Template Text Parser Templates

[Arista Networks](https://www.arista.com) has been doing a great deal of work on the [Arista Validated Design](https://github.com/aristanetworks/ansible-avd) which aims to generate configuration from structured data through a set of ansible playbooks and jinja2 templates.
[Template Text Parser](https://pypi.org/project/ttp/) is basically the inverse - it takes one or more templates, and attempts to generate structured data from the provided inputs

This is the beginning of a set of templates, using TTP, and the AVD data structures to try and generate compatible structured outputs that could later be used (if you wanted) to feed in through the AVD config generation.

# Notes
* This is not guaranteed to be directly AVD compatible at this time (Q4 2020), just reasonably close.
* The idealised goal is to generate structured data that is directly compatible with [AVD](https://github.com/aristanetworks/ansible-avd) - this means that in some areas, the template structure may be more convoluted then necessary, simply to bend the parser into delivering the desired outcome. This not an problem with the data-model, or the parser, simply the reality that we have to deal with - some post-processing may also be needed.
* The use of a "blank" match at the beginning of some templates is to set default parameters e.g.
```jinja
{{ mode | set("access") }}
```
reflects that the default mode of an Ethernet interface is access, but this needs to be inserted to conform to the  target data-model e.g.
```json
{
	"interfaces": {
		"ethernet": {
			"description": "WAN:23-5 circuit to Mosco",
			"ethernet_interface": "2/4",
			"mode": "access"
		}
	}
}
```
* If constructing a master template (single file) to capture the entire config, or even partial config, for a device, you should structure the order of the template groups to match the order that the config you are parsing, otherwise the parser gets out of sync and you end up with unexpected elements getting added e.g. do Port-Channel group before Ethernet, SVI, etc.
* If your output jinja templates expect a list, then you should explicity cast the group as such, otherwise if there is only a single match the result will be a dict, with would be undesirable e.g. we parse one or more static routes:
```jinja
<group name="static_routes*">
ip route {{ destination_address_prefix }}/{{ destination_address_prefix_mask }} {{next_hop | ORPHRASE}}
</group>
```
but our template explicitly expects this to be a list:
```jinja
{% if static_routes is defined and static_routes is not none %}
!
{%     for static_route in static_routes %}
ip route {% if static_route.vrf is defined %}vrf {{static_route.vrf}} {% endif %}{{ static_route.destination_address_prefix }}/{{ static_route.destination_address_prefix_mask }} {{static_route.next_hop}}
{%     endfor %}
{% endif %}
```

# Example

A template for [VLAN interfaces/SVIs](./templates/vlan-interfaces.ttp) as an example:

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
			"vlan_interfaces": {
				"246": {
					"ip_address": "10.10.20.2/23",
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

# Tips

## VSCODE Syntax Highlighting
* Install the [Better Jinja](https://marketplace.visualstudio.com/items?itemName=samuelcolvin.jinjahtml) extension, and associate .ttp files with *Jinja XML* language for my preferred syntax highlighting - thanks to brobare for that tip.