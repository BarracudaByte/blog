# What is SOAR?

![Confused Programmer](https://github.com/BarracudaByte/blog/blob/main/images/what-is-soar-header.png?raw=true)

Anyone working in Cyber Security has probably encountered at least once in their day to day an absolutely boring and repetitive task that could be better automated. For a lot of these tasks simple python or powershell scripts are enough, but sometimes you need some more funcionality, this is where **SOAR** comes into play. 

## What SOAR is and isn't

![Security Orechestration Automationa and Response](https://github.com/BarracudaByte/blog/blob/main/images/soar_00.png?raw=true)

As its name suggests (SOAR stands for **Security Orechestration, Automation, and Response**), SOAR is not only an automation tool but combines data of different tools and allows to create workflows to quicker to respond to incidents:

1. *Security Orechestration:* Make different cyber security tools work together seamlessly
2. *Automation:* Scripts to automate repetitive and time-consuming tasks
3. *Response*: Workflows to automatically respond to specific types of incidents

While SOAR tools have case management abilities, they do not do any sort of log management, event correlation, and real-time monitoring, which are usually all parts of a SIEM tool. 

![SIEM and SOAR](https://github.com/BarracudaByte/blog/blob/main/images/soar_01.png?raw=true)

## When to consider using a SOAR tool?

Most SOAR tools are not necessarily cheap and a lot of them don't even list their prices online. Still, it might be worth considering using one as it can save analyst a lot of time (and sanity). 

So what are things SOAR can help you with? Here are some ideas where SOAR can have a big impact:

- **case note enrichments**: detections provide analysts with the basic information they need to triage case, from their they use different tools to get a better understanding of the case, like looking up users in AD, checking a file hash in VirusTotal, and so on. SOAR can do all that once a case comes in and then provide a summarised view to the analyst
- **phishing analysis**: phishing emails are the most common cyber security incidents, therefore, automating part of this process can literally be a time saver. SOAR playbooks can parse the reported emails, submit any links to tools like URLScan.io or VirusTotal and send automatic email response to users. 
- **scheduled tasks**: no matter of pulling a report in a given interval or running a script every once in a while, SOAR tools have the ability to run jobs like these. What technically could be also done in a cron job, SOAR (at least the good ones) have the additional benefit of showing when a job failed, or setting it to automatically re-run. 

### Do you need coding experience to use SOAR?

Most SOAR tools advertise themselves as no-code solutions, which means they are basically designed for analysts with little to no coding experience. That being said, you can only go so far with the in-build building blocks, which are usually written in Python. So once that limit is reached it's good to know some coding to be able to write your own. 

On the other hand, experience programmers might get frustrated sometimes working in a SOAR tool. That is, because most SOAR tools being designed as no-code solutions and are, because of that, limiting what you can write yourself. 

## SOAR tools
Gartner has coined the term SOAR in 2015 and since then more and more SOAR tools have come to the market. Most of the major cyber security providers have nowadays their own SOAR tool:

- Cortex XSOAR (previous Demisto) by Palo Alto Networks
- Splunk Phantom
- QRadar by IBM
- Swimland Turbine
- Google SecOps SOAR

There are also some new kids on the block, with promising SOAR tools:
- The Hive by StrangeBee
- Tracecat (Open-Source)
- Tines

![Newcomer SOAR Tools](https://github.com/BarracudaByte/blog/blob/main/images/soar_02.png?raw=true)

There are also some SIEM tools that have some automation capabilities built in, which I have not listed here.

## Should you use SOAR?

As mentioned, most SOAR tools, especially the ones created by big companies are not cheap (I mean what cyber security tools really are?), so especially for smaller teams getting a SOAR tools might quickly exceed the buged limit. However, with open-source options like Tracecat this may not be an issue anymore. 

Budget aside, SOAR tools are fantastic for teams that are not too much into coding as most of them come with pre-build building blocks that allow you to quickly assemble a playbook. Those inbuild blocks can reach their limit sometimes very quickly, then it can be useful having someone with development experience on the team who can create custom ones.

In the end (as always) it comes down to your own requirements, your budget, and what exactly you are trying to automate. For example, if you just want to have a more reliable cron job, then Apache Workflow or something similar is probably a better choice simply from a money perspective. If you on the other hand want to decrease the time analysts spend on cases, SOAR may be just the right thing for you.