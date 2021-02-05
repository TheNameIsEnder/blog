+++

author = "Jacob Hell"
title = "What Are Reliability, Scalability, and Maintainability in Distributed Systems?"
date = "2021-02-05"
tags = [ "distributed", "data"]

+++

<!--more-->

## Reliability

Roughly, reliability means "working correctly" even if something goes wrong. This thing going wrong can be hardware faults, software faults, or human error. A synonym for reliability is fault-tolerant. It is impossible for a system to be completely fault tolerant. One tool you can use to test your system's tolerance is Netflix's [Chaos Monkey](https://netflix.github.io/chaosmonkey/).

### Hardware Faults

As we move into a cloud based style of software development, single machine failure is getting less common. However, cloud services sometimes randomly go out. One day, I was unable to use Azure due to the [outage in Texas](https://siliconangle.com/2018/09/04/severe-weather-impacts-microsofts-azure-cloud-services-texas/).

### Software Faults

Due to bugs, software issues can bring down entire systems. These issues can't be stopped completely, but can be mitigated with unit testing, integration testing, and monitoring system behavior in production.

### Human Error

Configuration error by humans is the leading causes of faults in systems. To mitigate this:

* Design systems to minimize opportunity for error.
* Provide separate instances of the system away from production for testing and exploring.
* Test using automated processes.
* Allow for quick and easy recovery from human errors.
* Setup monitoring for metrics and errors.



## Scalability

Scalability is how well a system can respond to load. Load is described with load feature terms. For example, Twitter must handle tweets at a rate of 12000 tweets per second.

In online systems, we usually care about the response time. Given 12000 tweets per second, how fast can a system respond to a user? If you store data on this, it is best to take the median of the response time.

To respond to increased load, you can either scale-up or scale-out. Scaling up means improving system performance by buying better equipment. Scaling out means to distribute the load horizontally.

There is no optimal best scaling design. It is dependent on the system.



## Maintainability

Refers to ongoing maintenance of a system. There's three design principles of software systems:

* Operability: Make it easy for teams to keep software running smoothly
  * Provide good monitoring of system internals
  * Provide good support for automation and integration of standard tools
  * Avoid dependency on individual machines
  * Provide good documentation
  * Provide good default behavior, but allow for privileged users to override
  * Allow for self healing
  * Minimize surprises
* Simplicity: Make it easy for new engineers to understand the system
  * Provide good abstraction
* Evolvability: Make it easy for new features to be added in future.
  * TDD and Agile

