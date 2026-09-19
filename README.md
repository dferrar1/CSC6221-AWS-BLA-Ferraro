# BLA#2 — AWS Networking & VPC

A three-part recorded walkthrough of the networking domain of the
**AWS Certified Solutions Architect – Associate (SAA-C03)** certification path.

**Author:** Dylan Ferraro
**Course:** *[course code / name]*
**Format:** Three videos, 5–8 minutes each (~23 minutes total)

---

## Videos

| # | Title | Runtime | Link |
|---|---|---|---|
| 1 | The Address Model | ~6:53 | *[paste link]* |
| 2 | Connectivity: In, Out, and Across | ~7:28 | *[paste link]* |
| 3 | Hybrid Connections, Security & Troubleshooting | ~7:58 | *[paste link]* |

*Playlist: [paste link]*

---

## Purpose

This BLA follows the **AWS certification path** option rather than the project
option. The goal was to work through the networking portion of the Solutions
Architect Associate curriculum and demonstrate that understanding by teaching it.

Studying notes lets you get away with recognizing a term. Explaining it to a
camera does not — you have to say *why* something exists, what breaks without it,
and when you'd choose it over the alternative. Recording these forced that gap
closed on a domain I had read several times but could not have taught.

The material is split across three videos instead of one because each segment
answers a genuinely different question, and because a 23-minute block is harder
to watch, harder to revisit, and harder to record cleanly than three focused
pieces.

---

## Why this topic matters

Every service you run in AWS sits inside a network you designed. An EC2
instance, an RDS database, a Lambda function reaching a private resource — all of
them depend on decisions made at the VPC layer.

What makes networking worth its own three videos is that **its failures don't
announce themselves as networking failures.** An application that can't reach its
database, a deployment that times out, a service that works in one availability
zone but not another — these present as application bugs. They are very often
routing, subnet, or firewall problems.

It's also the area where mistakes are most expensive to undo. A poorly chosen
address range can't be peered with anything that overlaps it, and the fix is
rebuilding the VPC. Most other architectural decisions in AWS are reversible.
This one effectively isn't.

---

## Video 1 — The Address Model

**What's introduced:** CIDR notation and subnet masking; the three private
address blocks reserved by IANA; VPC sizing rules and the constraint against
overlapping ranges; the relationship between a VPC, an Availability Zone, and a
subnet; the five IP addresses AWS reserves in every subnet; and how IPv6 behaves
differently in AWS.

**Why it matters:** This is the layer everything else is built on, and it's the
layer that can't be changed later. The video also settles a definition that most
material leaves implicit — **there is no "public" setting on a subnet.** A subnet
is public if and only if its route table has a route to an Internet Gateway.
Understanding that single point converts most of VPC from memorization into
reasoning, which is why it belongs in the first video rather than buried
alongside gateways.

The practical payoff is subnet sizing. Because AWS takes five addresses out of
every subnet, a `/27` provides 27 usable addresses, not 32 — enough to fail a
requirement for 29 hosts. That arithmetic appears on the exam and in real
capacity planning.

---

## Video 2 — Connectivity: In, Out, and Across

**What's introduced:** Route tables and the undeletable local route; Internet
Gateways; NAT instances and NAT Gateways, including why NAT must be deployed per
Availability Zone; the egress-only Internet Gateway for IPv6; bastion hosts; VPC
Endpoints in both gateway and interface form; AWS PrivateLink; and VPC Peering.

**Why it matters:** This is where the design work actually happens. The recurring
theme is that **private resources still need to reach things** — the internet for
patches, AWS services for storage, other VPCs for shared infrastructure — without
ever becoming reachable from outside. Each service in this video solves one
version of that problem, and choosing the wrong one costs either money or
security.

Two examples carry real consequences. Routing S3 traffic through a NAT Gateway
instead of a free Gateway Endpoint means paying per gigabyte to reach a service
that was already adjacent. And deploying a single NAT Gateway for a multi-AZ
workload creates a single point of failure in an architecture built specifically
to avoid one.

---

## Video 3 — Hybrid Connections, Security & Troubleshooting

**What's introduced:** Transit Gateway; Site-to-Site VPN with its Virtual Private
Gateway and Customer Gateway; AWS Direct Connect; Security Groups versus Network
ACLs; ephemeral ports; VPC Flow Logs; AWS Network Firewall; and VPC Traffic
Mirroring.

**Why it matters:** Two things come together here — connecting a VPC to
infrastructure outside AWS, and figuring out what went wrong when traffic doesn't
flow.

The security half centers on one distinction that explains a large share of
real-world connectivity bugs: **security groups are stateful and NACLs are
stateless.** A security group automatically permits the response to any request it
allowed in. A NACL does not, which is why a correctly configured inbound rule can
still produce connections that hang — the reply is being dropped on the way out.

That leads to the most useful moment in the series. If a VPC Flow Log shows a
request **accepted inbound but rejected outbound**, it is always the NACL, never
the security group — because a stateful security group cannot reject a response
it has already implicitly allowed. That's a deduction rather than a fact to
memorize, and it turns a long debugging session into a short one.

---

## AWS services covered

**Core:** Amazon VPC, subnets, route tables, Internet Gateway, NAT Gateway, NAT
instances, egress-only Internet Gateway, Elastic IP, ENI

**Connectivity:** VPC Endpoints (Gateway and Interface), AWS PrivateLink, VPC
Peering, AWS Transit Gateway, AWS Resource Access Manager, Site-to-Site VPN,
Virtual Private Gateway, Customer Gateway, AWS Direct Connect, Direct Connect
Gateway

**Security and monitoring:** Security Groups, Network ACLs, VPC Flow Logs, AWS
Network Firewall, AWS Firewall Manager, Gateway Load Balancer, VPC Traffic
Mirroring, Amazon CloudWatch Logs Insights, Amazon Athena

**Supporting concepts:** CIDR and subnet masking, RFC 1918 private address space,
IPv4 and IPv6 dual-stack, ephemeral port ranges, stateful vs stateless
inspection

---

## Key takeaways

1. **The route table is the network.** Public versus private, internet access,
   peering, endpoints — all of it is routing. It's the right first thing to check
   when something can't connect.
2. **Stateful versus stateless explains most connectivity bugs.** Security groups
   forgive an incomplete rule set; NACLs don't.
3. **Design the address space as if it's permanent**, because overlapping CIDRs
   can't be peered and can't be retrofitted.
4. **Layer the controls.** NACL at the subnet, security group at the instance,
   Network Firewall when you need to inspect what's actually inside the traffic.

---

## Notes on accuracy

Every figure stated in these videos was checked against current AWS documentation
before recording. That pass corrected four things carried in my original notes:

- The default VPC is **`172.31.0.0/16`**, not `172.16.0.0/12`
- Five CIDR blocks per VPC is the **default quota**, raisable to 50
- **ClassicLink no longer exists** — EC2-Classic was retired in August 2022
- Regional NAT Gateway was **excluded** rather than stated without verification

Corrections are welcome. If something here is wrong, I'd rather know.

---

## References

- [AWS Certified Solutions Architect – Associate (SAA-C03) Exam Guide](https://aws.amazon.com/certification/certified-solutions-architect-associate/)
- [Amazon VPC User Guide](https://docs.aws.amazon.com/vpc/latest/userguide/)
- [VPC Flow Logs](https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs.html)
- [Security Groups and Network ACLs](https://docs.aws.amazon.com/vpc/latest/userguide/infrastructure-security.html)
- [AWS Direct Connect User Guide](https://docs.aws.amazon.com/directconnect/latest/UserGuide/)
- [AWS Transit Gateway](https://docs.aws.amazon.com/vpc/latest/tgw/)
- [AWS PrivateLink](https://docs.aws.amazon.com/vpc/latest/privatelink/)
