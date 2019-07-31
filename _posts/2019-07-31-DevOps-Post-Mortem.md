---
title:      A little step towards writing incident postmortem to learn from previous history and avoid needless blaming
layout:     post
category:   Tutorial
tags: 	    [devops, sre, incident, postmortem, crash, system analysis]
#feature:   /assets/img/1_tsunami_udp_data_transfer.svg
---
Errors are sneaky, often remain hidden, smart and somehow get past no matter they are caused by any human action or any prevailing bug. This error often causes incidents in the running system causing a great negative impact on the team. To avoid the same mistakes in the future it is needed to document them properly which is often referred to as incident postmortem writing.
<!--more-->
As an operation/reliability engineer, you are already well introduced to service interruption and you need to shift your focus on quickly recovering the situation. What if you solve the problem and restore team discipline by solving this crisis in a short time? Do you think enough about mitigating the same problem again and again in the future? What if you forget the service recovery procedure over time by busywork or any team shift? Well, this is the time you offer yourself writing post mortem report which is a great blessing to your future self or any new engineer assigned for the same task.

So let's discuss on contents of postmortems. This report consists of some sections which are very specific to its purpose. Such as incident date, Author, Status, Summary, Impact, Root Cause, Trigger, Resolution, Detection, Proof of Concept for future mitigation, Supporting information and lastly people involved behind the solution of the crisis. This report can be brought to a greater extent. That depends on the size of the team, number of users and business value of the service. A sample post mortem report would look like,

```
# Date

July 29th, 2019

# Author

The postmortem report was written by:

DevOps Guy (devops@example.com)

# Status 

Resolved  

# Summary 

X application (hello.example.com) was down for 5 minutes 9 seconds. Due to SQL error in the database constraint in liquibase settings residing under source code. During the error time application was responding with HTTP error 404.

**Incident Occurrence Time:** 11:57 A.M. ( UTC + 6:00 )
**Investigation Starts:** 11:58 A.M. ( UTC + 6:00 )
**Investigation Finished:** 12:03 P.M. ( UTC + 6:00 )

Time Needed: 5 minutes

# Impact 

This outage impacted SRE team to test features on staging application. No real customer was harmed.

# Root Causes

Source code changes resulted in automated build which made liquibase to perform some changes in database but it was violating database constraint which resulted in database/backend error causing application to crash.

# Trigger 

The outage was caused after the push to source code with commit id xxxxxxx. This commit eventually built S3 bucket build artifact with version xxxxxxxxxxxxxxxxxxxxxxxx. New artifact got deployed into server and occurred outage.

# Resolution 

The outage was fixed after the push to source code with commit id yyyyyyy.  This commit eventually built S3 bucket build artifact with version yyyyyyyyyyyyyyyyyyyyyyy. New artifact got deployed and fixed outage. 

# Detection 

The problem was noticed by team SRE first. Later we got automated crash report in slack with email from our monitoring agent X{OpsGenine / PagerDuty / Uptime Robot}.

# PoC for Future Mitigation

* Need to implement database version controlling with application specific branching
* Need to implement blue green deployment in application
* Need to reduce build time to deliver software earlier

# Supporting Information

Additional screenshot as a document of proof:

* Any screenshot of your alerting agent here.

# Names of People Involved

People who worked behind resolving the issue:

 * Backend Guy ( backend@example.com )
 * DevOps Guy ( devops@example.com )

```

Wait, you might be thinking I am blabbering too much about reporting, postmortems, procedures, etc. You might be thinking "how does blame is related to this?" Let me explain that a little. Remember that a good team is better than the best when good intentions of team members are appreciated in parallel with their hard work. However, by nature, we are prone to error and in a team perspective, this might somehow result in chaos. During the time of service downtime crisis, the meaningful approaches would be covering up mistakes, bringing the discipline back and documenting the mistakes for better learning and simulating for future avoidance. And through writing postmortems, it is possible to bring out the maximum value of the team in the future by the removal of casting blame to one another. So say no to repeated mistakes!