# Network Segmentation Lab: Staff, Servers and Guests

A Cisco Packet Tracer lab that splits one network into three subnets and uses an access list to isolate the Guest network.

## Why I built it
My lecturer asked about network segmentation in class, so I built a lab to show it working. A company's guest Wi-Fi is a risk because anyone can connect to it. I split one network into Staff, Servers and Guests and wrote an access list that stops Guests from reaching the Servers and Staff. This limits the damage if a guest device is compromised.

## Design
Starting network: 192.168.1.0/24, split into three /27 subnets (mask 255.255.255.224, 30 usable hosts each). One router connects them. Each router port is the gateway for its subnet.

| Network | Subnet | Gateway | Usable hosts | Broadcast |
|---|---|---|---|---|
| Staff | 192.168.1.0/27 | 192.168.1.1 | .1 to .30 | .31 |
| Servers | 192.168.1.32/27 | 192.168.1.33 | .33 to .62 | .63 |
| Guests | 192.168.1.64/27 | 192.168.1.65 | .65 to .94 | .95 |

Test PCs: Staff 192.168.1.2, Servers 192.168.1.34, Guests 192.168.1.66.

## Security rule
Applied inbound on the Guests port (GigabitEthernet0/2):

```
10 deny ip 192.168.1.64 0.0.0.31 192.168.1.32 0.0.0.31
15 deny ip 192.168.1.64 0.0.0.31 192.168.1.0 0.0.0.31
20 permit ip any any
```

The deny lines come before the permit because the router stops at the first match.

## Proof (screenshots attached)
Before the rule, Guests reached Servers (4 of 4 replies, TTL 127).

After the rule:
1. Guests to Servers: failed, unreachable reply from 192.168.1.65
2. Guests to Staff: failed, same reply
3. Guests to their own gateway: worked
4. Staff to Servers: worked

## What I learned
1. I first entered the Staff port mask as 255.255.255.0 instead of 255.255.255.224. That would tell the router Staff owns the whole /24 and overlap the other subnets. The router's command log showed it, so I corrected the mask and checked it again.
2. My Staff deny line first landed below the permit, so Guests could still ping Staff. I saw it in the access list output, deleted that line and re-added it above the permit. A rule that is typed in is not a rule that works, so I test every change with pings.

## Limitation
Guests can still reach their own gateway and anything not blocked. A real design would also restrict Guests to internet access only.
