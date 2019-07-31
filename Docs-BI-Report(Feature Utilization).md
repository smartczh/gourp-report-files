# Docs BI Report (Feature Utilization)

## Basic Requirements

+ Single location => One storage
+ Understand features => Visualization
+ Open to new features implemented => Extensibility
+ near real-time and historical views => Fast data synchronization and permanent storage

More details in this [spec](https://review.docs.microsoft.com/en-us/new-hope/specs/ops/docs-feature-utilization?branch=master).

## Data Storage

Azure data explorer (Kusto) is chosen now for following reasons compared to other storages:

+ Kusto engine compared with traditional SQL Server:
  + Column-oriented DBMS => OLAP-focused
  + All columns indexing automatically => Fast
  + Support Structured, Semi-Sturctured and Unstructured data from
+ Design for analyzing large volumes of logs or data:
  + Read, append-many
  + Delete, update almost never
+ Nearly permanent storage (10 years), where Application Insights (AI) is 90 days.
+ Supported by powerBI as data source

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
Event Hubs (7 days) < Application Insights (90 days) < Persistent storage (configurable) <= Logs
Exposing API depends on its source (high dependency).
Azure Data Factory (ADF) supports persistent storage more.

## Ways to Move Data from Sources to Our Storage

### AI to ADE vs Event Hubs to ADE vs Custom Libs to ADE

This comparison makes sense when we need data generated from the process, which is not persistent.

Name | Source Effect | Ingest Ways| Advantages | limitation |
--- | --- | --- | --- | ---
AI | Write AI Track code | REST API; Continuous export<br>[more info](#AI-ingest-way) | provide rich metrics functions for data analysis (act like a small BI mid-platform based on the data in it) | not persistent (90 days); <br> pull mode of ingestion
Event Hubs | Write code to send event | ADE build-in support;<br>event-based azure function;<br>stream analysis;<br>ADF<br>[more info](#event-hubs-ingest-way) | push mode; scalability of message consumers | Short message Retention (7 days);<br>No data analytical ability
Custom libs | import our lib and invoke it | directly writing via .NET Standard SDK  |directly writing | Incurs development overhead since the application for custom ingestion must **handle errors** and **ensure data consistency**;<br> May **affect performance**

#### AI to ADE Ways {#AI-ingest-way}

AI supports two ways to export data.

+ REST API, can be invoked by: **suggested**
  + ADF tools (tumbling window triggered), **suggested**
  + Azure Function (schedule triggered)
  + ...
+ Continuous export to blob logs, then read blog by
  + ADF tools
  + Azure Function
  + ...

#### Event Hubs to ADE Ways {#event-hubs-ingest-way}

+ Kusto build-in support (can't fetch eventData.properties info) **suggested**
+ Event-based Azure Function
+ ADF tools
+ Stream analysis 

## Design Thinking

Point | reason
--- | ---

### Data Process 

### Table Schema


the more customized, the more bug
the more code-free, the less customized
balanced
