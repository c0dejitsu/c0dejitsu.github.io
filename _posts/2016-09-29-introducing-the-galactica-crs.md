---
layout: post
title:  "Introducing the Galactica CRS"
date:   2016-09-29 08:00:00
categories: galactica
---

Team CodeJitsu is proud to introduce you to the Galactica CRS! In the
coming days, we will publish a series of blog posts about our
technology, which earned the fifth place in DARPA’s [*Cyber Grand
Challenge*](https://www.cybergrandchallenge.com/).

Galactica leads a rag-tag fleet of powerful symbolic execution engines,
binary instrumentation tools and fuzzers on a heroic quest to find
cybersecurity for mankind. We, Galactica’s creators, would like you to
be part of the quest: explore the technology inside the CRS, ask
questions using the comment box below, and try out our open-source
releases as we announce them in the coming days.

<figure style="float: left; width: 49%;">
  <img src="{{"/static/images/galactica_crs.jpg" | prepend: site.baseurl}}" alt="The Galactica CRS"/>
  <figcaption>
  The Galactica CRS
  </figcaption>
</figure>
<figure style="float: right; width: 49%;">
  <img src="{{"/static/images/codejitsu_team.jpg" | prepend: site.baseurl}}" alt="CodeJitsu, creators of Galactica"/>
  <figcaption>
  ... and its creators, team CodeJitsu.
  </figcaption>
</figure>


{: .clearfix }
The fleet
---------

To start, today’s post gives an overview of the entire system.

<figure>
  <img src="{{"/static/images/galactica_components.png" | prepend: site.baseurl}}" alt="An overview of Galactica's components"/>
</figure>

The figure above shows the components that together constitute the
Galactica CRS. These are the parts:

-   Everything is orchestrated by the *CodeJitsu REST interface.* This
    component is responsible for talking with DARPA through the team
    interface, keeping track of the competition and of each challenge
    program, and dispatching tasks through the scheduler.

-   Our *scheduler* is a custom Mesos framework that decides where each
    task should be executed. It ensures that big long tasks (like
    symbolically executing a program on 40 cores) get scheduled
    efficiently along with small short-running tasks. Our scheduler is
    also fault-tolerant, and restarts tasks if nodes fail.

-   The tasks and the CodeJitsu interface share state in a central
    *database* and a *shared file system*. The file system stores
    binaries, testcases, program inputs gathered from the network. The
    database provides atomicity and durability for all the other small
    bits of state.

-   Galactica processes challenge programs using a number of tasks. The
    first of these is *disassembly,* which reconstructs the
    control-flow-graph (CFG).

-   Galactica then subjects the challenge program to *static analysis*
    using [*RevGen*](http://dslab.epfl.ch/pubs/revgen.pdf)…

-   … *symbolic execution* using [*S2E*](http://s2e.epfl.ch/)…

-   … and *fuzz testing* using [*AFL*](http://lcamtuf.coredump.cx/afl/).

-   To protect programs against attacks from other teams, the *BinKungfu
    binary hardening engine* creates several hardened versions of
    the program. These differ in performance, security,
    and functionality. BinKungfu was written from scratch for CGC.

-   The *performance testing* component analyzes each of the hardened
    program versions. It measures how well they perform, and detects
    security/functionality issues. The component reports performance
    data back to the REST interface, which then selects the best
    version to submit.

-   *Traffic replay* captures the network traffic from service polls
    and attacks. It replays that traffic against the original program
    and against the hardened version. The traffic replay component can
    detect attacks, as well as generate test inputs for the
    performance testing component.

-   Last but not least, our *watchdog* based on
    [*Monit*](https://mmonit.com/monit/) monitors all
    these components. If anything stops working, it uses custom
    service checks and service dependencies to figure out where the
    problem may lie. The watchdog then restarts individual tasks or
    reboots entire nodes.

We love statistics :) Here are some about the CRS as a whole:

{: .bordered .highlight }
**Component**              | **Built using**                | **Stats**
-------------------------- | ------------------------------ | -------------------------
CodeJitsu REST interface   | Python (based on Django)       | 20.0 KLOC
Database                   | Postgres                       | FIXME records, FIXME MB
Shared file system         | GlusterFS                      | FIXME GB
Scheduler                  | Apache Mesos, Python           | 2.8 KLOC
Disassembly                | IDA Pro, Python                | 1.3 KLOC
Static Analysis            | RevGen                         | 2.3 KLOC
Symbolic Execution         | S2E                            | 50.0 KLOC
Fuzz testing               | AFL, C                         | 1.1 KLOC
Fuzz interface             | Python                         | 1.2 KLOC
BinKungfu hagrdening       | Python, Capstone, pyelftools   | 9.3 KLOC
Performance testing        | Python, QEMU                   | 2.5 KLOC
Traffic replay             | Python, C++                    | 5.8 KLOC
Watchdog                   | Monit, custom check scripts    | 0.5 KLOC

Lines of code are source lines created by CodeJitsu, not counting existing code,
comments, and blanks.

In the coming days, we will present these components in detail in their
own blog posts. In the meantime, let us know what you’re most interested
in! Post your questions, comments and requests in the comment section
below. We promise to answer.

Stay tuned for the next posts! – [Jonas](https://people.epfl.ch/jonas.wagner) (Team CodeJitsu)
