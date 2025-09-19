# BGP (Border Gateway Protocol) Fundamentals

BGP is the protocol that makes the internet work by routing traffic between different networks. Understanding BGP is crucial for SRE professionals working with network infrastructure, cloud connectivity, and traffic engineering.

## Overview

**BGP (Border Gateway Protocol)** is the routing protocol used to exchange routing information between different autonomous systems (AS) on the internet. It's often called the "protocol of the internet" because it determines how data flows between networks globally.

## Key Concepts

### Autonomous System (AS)
- A network or group of networks under one administrative control
- Examples: ISPs, cloud providers, large enterprises
- Each AS has a unique AS Number (ASN)
- ASNs range from 1 to 4,294,967,295 (32-bit)

### BGP Session Types
1. **eBGP (External BGP)**: Between different autonomous systems
2. **iBGP (Internal BGP)**: Within the same autonomous system

## How BGP Works

### Basic BGP Flow
```
AS 100 (ISP A) <--eBGP--> AS 200 (ISP B) <--eBGP--> AS 300 (Cloud Provider)
     |                         |                         |
   iBGP                      iBGP                      iBGP
     |                         |                         |
Router A1                  Router B1                 Router C1
Router A2                  Router B2                 Router C2
```

### BGP Route Advertisement Process
```
1. Router learns about a network (directly connected or via IGP)
     |
2. Router announces this network to BGP neighbors
     |
3. BGP neighbors evaluate the route using path attributes
     |
4. Best route is selected and installed in routing table
     |
5. Route is advertised to other BGP neighbors (with policy filters)
```

## BGP Path Attributes

BGP uses various attributes to determine the best path to a destination:

### 1. AS_PATH
- **Purpose**: Lists all AS numbers the route has traversed
- **Selection**: Shorter AS_PATH is preferred
- **Loop Prevention**: Prevents routing loops by rejecting routes containing own ASN

### 2. NEXT_HOP
- **Purpose**: IP address of the next router to reach the destination
- **eBGP**: Set to the IP of the advertising router
- **iBGP**: Usually unchanged from original eBGP advertisement

### 3. LOCAL_PREF (Local Preference)
- **Purpose**: Indicates preferred path for outbound traffic
- **Scope**: Local to AS (not advertised to external peers)
- **Selection**: Higher value is preferred

### 4. MED (Multi-Exit Discriminator)
- **Purpose**: Suggests preferred entry point into AS
- **Scope**: Advertised to neighboring AS
- **Selection**: Lower value is preferred

### 5. ORIGIN
- **Purpose**: How the route was introduced into BGP
- **Values**: 
  - IGP (i): Route learned from interior gateway protocol
  - EGP (e): Route learned from exterior gateway protocol  
  - Incomplete (?): Route redistributed from other sources

## BGP Route Selection Process

BGP uses a deterministic process to select the best route:

```
1. Highest Weight (Cisco proprietary)
     |
2. Highest Local Preference
     |
3. Locally originated routes (next-hop 0.0.0.0)
     |
4. Shortest AS_PATH
     |
5. Lowest ORIGIN (IGP < EGP < Incomplete)
     |
6. Lowest MED (if from same neighboring AS)
     |
7. eBGP over iBGP
     |
8. Lowest IGP metric to next-hop
     |
9. Oldest route (for stability)
     |
10. Lowest Router ID
     |
11. Lowest cluster list length
     |
12. Lowest neighbor IP address
```

## BGP in Cloud and SRE Context

### 1. Cloud Provider Connectivity

#### AWS Direct Connect
```
Customer Network (AS 65000)
     |
     |---> Direct Connect Gateway
              |
              |---> AWS (AS 7224)
                      |
                      |---> VPC (Virtual Private Gateway)
```

#### Multi-Cloud BGP Setup
```
Corporate Network (AS 65001)
     |
     |---> ISP (AS 100) ---> Internet
     |
     |---> AWS Direct Connect (AS 7224)
     |
     |---> Azure ExpressRoute (AS 12076)
     |
     |---> GCP Cloud Interconnect (AS 15169)
```

### 2. Anycast Implementation

BGP enables Anycast by allowing multiple locations to advertise the same IP prefix:

```
DNS Server Network using Anycast:

Location A (AS 65100): Advertises 203.0.113.0/24
Location B (AS 65100): Advertises 203.0.113.0/24  
Location C (AS 65100): Advertises 203.0.113.0/24

Client Traffic:
- Clients in Region A → routed to Location A
- Clients in Region B → routed to Location B  
- Clients in Region C → routed to Location C
```

### 3. Traffic Engineering

#### Inbound Traffic Control
```
# Prefer specific path for inbound traffic using AS_PATH prepending
AS 65001 announces:
- To Provider A: 203.0.113.0/24 (normal announcement)
- To Provider B: 203.0.113.0/24 with AS_PATH 65001 65001 65001
```

#### Outbound Traffic Control
```
# Use Local Preference to control outbound traffic
route-map PREFER_PROVIDER_A permit 10
 set local-preference 200
route-map PREFER_PROVIDER_B permit 10  
 set local-preference 100
```

## BGP Security Considerations

### 1. Route Hijacking
**Problem**: Malicious AS announces IP prefixes it doesn't own
**Mitigation**:
- RPKI (Resource Public Key Infrastructure)
- Route filtering based on IRR (Internet Routing Registry)
- BGP monitoring and alerting

### 2. Route Leaks
**Problem**: Unintended advertisement of routes to unintended peers
**Mitigation**:
- Proper route filtering policies
- AS_PATH filtering
- Community-based policies

### 3. BGP Monitoring
```bash
# Monitor BGP sessions
show ip bgp summary
show ip bgp neighbors

# Check BGP table
show ip bgp
show ip route bgp

# Monitor specific prefixes
show ip bgp 203.0.113.0/24
```

## Common BGP Issues and Troubleshooting

### 1. BGP Session Down

**Symptoms**:
- Routes not being learned/advertised
- Connectivity issues

**Troubleshooting Steps**:
```bash
# Check BGP neighbor status
show ip bgp neighbors 192.168.1.1

# Verify TCP connectivity
telnet 192.168.1.1 179

# Check routing to BGP peer
show ip route 192.168.1.1

# Review BGP logs
show logging | include BGP
```

### 2. Asymmetric Routing

**Problem**: Outbound and inbound traffic taking different paths

**Diagnosis**:
```bash
# Trace outbound path
traceroute 8.8.8.8

# Check BGP best path selection
show ip bgp 8.8.8.0/24

# Verify return path (requires cooperation from remote end)
```

**Solutions**:
- Adjust Local Preference for outbound control
- Use AS_PATH prepending for inbound control
- Implement consistent routing policies

### 3. Route Flapping

**Problem**: Routes being withdrawn and re-announced frequently

**Detection**:
```bash
# Check BGP dampening statistics
show ip bgp dampening flap-statistics

# Monitor route changes
show ip bgp neighbors 192.168.1.1 received-routes | begin 203.0.113.0
```

**Mitigation**:
- Implement BGP dampening
- Identify and fix root cause (link instability, configuration issues)
- Use route aggregation to reduce flapping impact

## BGP Best Practices for SRE

### 1. Monitoring and Alerting
- Monitor BGP session states
- Alert on route table size changes
- Track route convergence times
- Monitor for route hijacking

### 2. Configuration Management
- Use consistent routing policies
- Implement proper route filtering
- Document AS_PATH and community usage
- Regular configuration backups

### 3. Capacity Planning
- Monitor BGP table growth
- Plan for memory and CPU requirements
- Consider route aggregation strategies

### 4. Disaster Recovery
- Maintain redundant BGP sessions
- Test failover scenarios
- Document emergency procedures
- Keep contact information for upstream providers

## Interview Questions & Answers

### Q1: How does BGP prevent routing loops?
**Answer**: BGP prevents routing loops primarily through the AS_PATH attribute. When a router receives a BGP update, it checks if its own AS number is already in the AS_PATH. If it finds its ASN in the path, it rejects the route, preventing the loop. Additionally, iBGP uses split-horizon rules and requires full mesh or route reflectors to prevent loops within an AS.

### Q2: What's the difference between eBGP and iBGP?
**Answer**: 
- **eBGP**: Runs between different autonomous systems, decrements TTL, changes next-hop, and has an administrative distance of 20
- **iBGP**: Runs within the same AS, doesn't change TTL or next-hop by default, has administrative distance of 200, and requires full mesh or route reflectors to prevent loops

### Q3: How would you implement traffic engineering with BGP?
**Answer**: Traffic engineering with BGP involves controlling both inbound and outbound traffic:
- **Outbound**: Use Local Preference to prefer certain paths for traffic leaving your AS
- **Inbound**: Use AS_PATH prepending, MED adjustment, or selective announcements to influence how others reach you
- **Both**: Use communities for policy coordination and implement proper route filtering

### Q4: What is BGP convergence and why is it important?
**Answer**: BGP convergence is the time it takes for all routers to agree on the best paths after a topology change. It's important because during convergence, some traffic may be dropped or take suboptimal paths. Factors affecting convergence include BGP timers, route reflector design, and network size. SREs should monitor convergence times and optimize BGP design for faster convergence.

## Hands-On Exercises

### Exercise 1: BGP Route Analysis
```bash
# Analyze BGP table
show ip bgp | include 0.0.0.0
show ip bgp summary

# Check specific route
show ip bgp 8.8.8.0/24
show ip bgp 8.8.8.0/24 bestpath
```

### Exercise 2: BGP Path Manipulation
```bash
# Configure AS_PATH prepending
route-map PREPEND_PATH permit 10
 set as-path prepend 65001 65001

# Apply to neighbor
neighbor 192.168.1.1 route-map PREPEND_PATH out
```

### Exercise 3: BGP Monitoring Setup
```bash
# Enable BGP logging
router bgp 65001
 bgp log-neighbor-changes

# Monitor BGP events
show logging | include BGP
show ip bgp neighbors 192.168.1.1 flap-statistics
```

## References

- [RFC 4271: A Border Gateway Protocol 4 (BGP-4)](https://tools.ietf.org/html/rfc4271)
- [RFC 4760: Multiprotocol Extensions for BGP-4](https://tools.ietf.org/html/rfc4760)
- [BGP Security Best Practices](https://www.manrs.org/)
- [RIPE NCC BGP Training](https://www.ripe.net/support/training)

---

*This guide provides comprehensive coverage of BGP fundamentals essential for SRE professionals working with network infrastructure and traffic engineering.*
