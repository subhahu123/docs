# High Level System Design

* [Guide](https://www.hellointerview.com/learn/system-design/in-a-hurry/delivery)

Structure:

1. Requirements - Functional & Non functional
2. Core entities
3. API
4. High-level design
5. Deep dive

## Requirements

### Functional requirements

User should be able to...

### Non-functional requirements

> At this point I know many candidates might make estimates at specific numbers such as the amount of data per second etc. I'm going to keep this in mind and I'm going to come back to non-functional requirements and define numbers if it's going to influence my design.
>
> For now I'm just going to do high level non-functional requirements

IF clear, identify them, otherwise return later.

1. CAP Theorem: Should your system prioritize consistency or availability? Note, partition tolerance is a given in distributed systems.
2. Environment Constraints: Are there any constraints on the environment in which your system will run? For example, are you running on a mobile device with limited battery life? Running on devices with limited memory or limited bandwidth (e.g. streaming video on 3G)?
3. Scalability: All systems need to scale, but does this system have unique scaling requirements? For example, does it have bursty traffic at a specific time of day? Are there events, like holidays, that will cause a significant increase in traffic? Also consider the read vs write ratio here. Does your system need to scale reads or writes more?
4. Latency: How quickly does the system need to respond to user requests? Specifically consider any requests that require meaningful computation. For example, low latency search when designing Yelp.
5. Durability: How important is it that the data in your system is not lost?
6. Security: How secure does the system need to be? Consider data protection, access control, and compliance with regulations.
7. Fault Tolerance: How well does the system need to handle failures? Consider redundancy, failover, and recovery mechanisms.
8. Compliance

---

Out of scope

Ask interviewer if out of scope is reasonable

## Core Entities


