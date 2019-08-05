# Docs BI Report (Feature Utilization)

## Basic Requirements

+ Single location => One storage
+ Understand features => Visualization
+ Open to new features implemented => Extensibility
+ Near real-time and historical views => Fast data synchronization and permanent storage
+ from the view of the whole platform, per buildID, repoID, userID and group of them => Table schema, query-enabled

More details in this [spec](https://review.docs.microsoft.com/en-us/new-hope/specs/ops/docs-feature-utilization?branch=master).

## Data Storage

Azure data explorer (Kusto) is chosen now for following reasons compared to other storages:

+ Kusto engine compared with traditional SQL Server:
  + Column-oriented DBMS => OLAP-focused
  + All columns indexing automatically => Fast
  + Support Structured, Semi-Sturctured and Unstructured data from => integrating data
+ Design for analyzing large volumes of logs or data:
  + Read, append-many
  + Delete, update almost never
+ Nearly permanent storage (10 years), where Application Insights (AI) is 90 days.
+ Supported by powerBI as a data source

## Data Sources

We have following data sources Currently:
From the view of functions:

1. Feature Utilization related:
    + Application Insights (Docfx)
    + Event Hubs (OPBuild)
    + ...

2. BI:
    + Exposing API
    + Cosmos DB
    + Blog logs
    + ...

>[!NOTE]
>Thinking: Data Retention
Event Hubs (7 days) < Application Insights (90 days) < Permanent storage (configurable) <= Logs
Exposing API depends on its source (high dependency).
Azure Data Factory (ADF) supports permanent storage more.

## Ways to Move Data from Sources to Our Storage

### AI to ADE vs Event Hubs to ADE vs Custom Libs to ADE

When we need data generated from the process, which is not persistent.

Name | Source Effect | Ingest Ways| Advantages | limitation |
--- | --- | --- | --- | ---
AI | Write AI Track code;<br>Need a schema | REST API;<br> Continuous export <br> [more info](#AI-ingest-way) | Provide rich metrics functions for data analysis (act like a small BI mid-platform based on the data in it); <br> Force the source to generate Kusto-query-enabled data forms;<br> Support [cross product queries](https://docs.microsoft.com/en-us/azure/data-explorer/query-monitor-data) (preview);<br>Scalability of message consumers | not persistent (90 days); <br> pull mode of ingestion
Event Hubs | Write code to send event;<br>Need a schema | ADE build-in support;<br>Event-based Azure Function;<br>Stream Analysis;<br>ADF<br>[more info](#event-hubs-ingest-way) | Push mode;<br>Scalability of message consumers | Short message Retention (7 days);<br>No data analytical ability
Custom libs | Import our lib and use it | Directly writing via .NET Standard SDK  |Directly writing | Incurs development overhead since the application for custom ingestion must **handle errors** and **ensure data consistency**; <br> May **affect performance**

#### AI to ADE Ways {#AI-ingest-way}

AI supports [two ways](https://docs.microsoft.com/zh-cn/azure/azure-monitor/app/export-telemetry) to export data.

+ REST API, can be invoked by:
  + ADF tools (tumbling window triggered, minimal 15 mins)
  + Azure Function (schedule triggered)
  + Azure WebJobs
  + ...
+ Continuous export to blob logs, then read blog by
  + ADE build-in support
  + ADF tools
  + Azure Function
  + ...

>[!NOTE]
> Shortcomings of both
> + Rest API: 
>    + May overload the system
>    + Has built in throttling mechanisms (e.g., data size limitation)
> + Continuous export:
>    + Built-in format
>    + Hard to operate custom transformation (e.g., jsonPath to get keys)

#### Event Hubs to ADE Ways {#event-hubs-ingest-way}

Two sources, from event or capturing event log.

Name | Advantage | Limitation
----- | ---- | ----
[Kusto build-in support](https://docs.microsoft.com/zh-cn/azure/data-explorer/ingest-data-event-hub) | Code-free; <br> Robust | Can't fetch info in **eventData.properties** <br> Hard to customize logic
Event-based Azure Function | Custom logic;<br> Automatic scale | development overhead to **handle errors** and **ensure data consistency** (Kusto .NET sdk can help)
ADF tools | Integration | Do not support Event Hubs directly; <br> Read data from capturing blob logs
Stream Analysis | High integration with Event Hubs | Do not support direct output to ADE, need middle layer

## Design Thinking

### Points

+ We need all filtered raw data to achieve extendability to new features and less dependency on source side.
+ ADF is suitable when:
  + there are multiple supported data sources, whether in a pipeline or the project
  + configurations play an important role
+ BI report is our endpoint visualization, if the performance is unsatisfactory duo to front-end BI or Kusto query, data processing logic (like join query) should be moved forward.
+ Push mode is not necessary, since:
  + BI does not need such high timeliness.
  + Some processes might violate the principle of push mode (writing logs, Kusto update policy, Kusto queue writing sdk).
  + For some permanent storages, it is hard to realize push mode.

### Data processing places

+ Source sides when tracking traces or sending events (filter)
+ ADF pipelines or custom codes when moving data (filter and preprocessing)
+ Kusto function, update policy, query (generating indicators raw data)
+ BI query (there are many limitations in DirectQuery mode) (generating indicators)

We should balance the data processing logics in above places.

>[!NOTE]
> Pay attention to size limit in every process.

### Table Schema

+ Table source
  + Single: a source requires a table
    + join in query: may affect performance
    + Create a joined table using Kusto function and update policy: need handle data consistency
  + Multiple:
    + Need handle integrating logic
+ Link keys (if needed): buildID, repoID, userID
+ Dynamic type for custom dimensions
+ ...

### Comparison with existing BI

Name | Current | Our Goal
--- | --- | ---
Data ingestion frequency | Every 1 day | push mode or at least every 30 mins
Data ingestion ways | Source exposing API and WebJobs pull (high dependency) | ADF tools or raw data storage (less dependency)
BI report speed | BI from Traditional SQL => slow | BI from Kusto => fast

### Things to balance

+ Custom data processing && Robust process
+ Integration (unified tool, schema) && Feasibility in different environments
+ Fault-tolerant
+ Logic (work) placement: source side && our side
+ Existing works
  + Breaking (e.g., Event Hubs consumers)
  + Reusing (e.g., exposing API)

## Current work

+ Build BI：
+ Docfx BI:
+ Feature Utilization:
