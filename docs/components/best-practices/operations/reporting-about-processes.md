---
title:  Reporting about processes
tags:
    - Reporting
    - History
    - DWH
    - BI
    - KPI
    - SQL
    - MIS (Management Information System)
---

The Camunda engine automatically collects audit information about historical process or decision instances. Leverage this data by generating and displaying business relevant reports. Add business relevant phases and milestones to your process models serving as a basis for key performance indicators (KPIs).


## Modeling key performance indicators (KPIs)

When modeling a process, you always add information about important key performance indicators implicitly, like by introducing *start and end events*.

On top of that you can explicitly add

- Meaningful additional business *milestones* by modeling e.g. *intermediate events*. This might not have any execution semantics other than to leave a trace in the history of the workflow engine. The milestone is met as soon as the process has passed the event. Its status can therefore be *passed* or *not passed*.

- Meaningful business *phases* by modeling things like (embedded) *sub processes*. In contrast to a milestone, a phase's state can be *not entered*, currently *active* or *passed*.




Consider the following example - a "Tweet Approval Process" showing start and end events as well as *milestones*:

<div bpmn="reporting-about-processes-assets/TwitterReportingMilestone.bpmn" callouts="end_event_review_done_slowly,intermediate_event_tweet_approved,end_event_tweet_published,end_event_tweet_unpublished" />"

<span className="callout">3</span>

After one business day, the reviewer is reminded to speed up - and such reviews are internally *marked* by passing the end *event 'Review done slowly'*.

<span className="callout">4</span>

*Approved tweets* will pass the additional *intermediate event*. The *cycle time* up till that point is automatically captured too.

<span className="callout">5</span>

Furthermore, when tweets are successfully published, we are interested in the *ratio* of those tweets...

<span className="callout">6</span>

...when being compared to tweets that do not get published. Therefore, we model *two different end events* representing those two business end states of the process. Note that duplicate tweets will be *not published* even though they have been *approved* before. The more precisely we describe and *name* the business semantics of events, the better our KPI's will reflect the reality we want to measure!


When you do not (only) want to concentrate on milestones, but *phases* in your process, model the phases as sub processes:

<div bpmn="reporting-about-processes-assets/TwitterReportingPhases.bpmn" callouts="sub_process_review,sub_process_publication" />


<span className="callout">1</span>

The phase *Review* - modeled with a sub process - will be active while the human reviewer will need to find time to complete the task...

<span className="callout">2</span>

... whereas the phase *Publication* will be completed automatically - hence process instances "remaining" there for longer than a few seconds will probably indicate ongoing problems with the uptime and reachability of the used services.




## History architecture

It is useful to understand the architecture around history data in Camunda Cloud. 

:::caution Camunda Platform 7
Note that the history architecture is very different in Camunda Platform 7.x, see [Camunda Platform 7 User Guide](https://docs.camunda.org/manual/latest/user-guide/process-engine/history/).
:::


![History architecture](reporting-about-processes-assets/history-architecture.png)

Camunda saves historical data not just when a process instance finishes, but on the go - while a process instance is active. By doing this Camunda separates runtime data from history data - a growing history will not influence the runtime behavior: you should never need to access runtime data for reporting purposes.

Historical data can be leveraged via three possible mechanisms:

- *Camunda tools*: You can simply leverage Camunda Operate or Camunda Optimize. This is a very simple approach, that works out-of-the-box and should satisfy many requirements already. Camunda Operarte focuses on operational use cases ("where is my process? Why did this fail?") whereas Camunda Optimize provides business intelligence about your processes. Optimize allows you to build reports and dashboards including setting alerts for thresholds.

- *Query API*: Using the public API (**currently under development**). This has the advantage that you can make use of the history data within your own applications. 

- Pushing *events*: Pushing Camunda events by using [exporters](/docs/components/zeebe/technical-concepts/architecture/#exporter). Note that you can only add own exporters in a self-managed setting, not in Camunda Cloud SaaS. Exporters have the advantage that you can push the data into any infrastructure you have, and possibly even filter or enrich the data in that step.



## Connecting custom business intelligence systems (BI), data warehouses (DWH), or monitoring solutions

You might move data from the Camunda History to a decoupled system like a Business Intelligence (BI) solution, a Data Warehouse (DWH), some Data Lake, or a own monitoring solution, for example based on Prometheus.

Leveraging typical BI system's *ETL* (extract, transform and load) features allows to optimize data structure for your reporting purposes (to *speed up* report generation) or to combine generic process engine data with business entities (to allow for *more in-depth analysis*).

To get the data into the BI system, you can leverage one of the mechanisms described above. Our recommendation genrally is:

- In a self-managed setting, implement a custom exporter.
- In SaaS, leverage the history API to regularly pull data, as custom exporters are not supported there.