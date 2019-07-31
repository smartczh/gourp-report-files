# Docs Feature Utilization Designs Report

## Basic Requirements

+ Single location => One storage
+ Understand features => Visualization
+ Open to new features implemented => Extensibility
+ near real-time and historical views => Fast data synchronization and permanent storage

More detail in this [spec](https://review.docs.microsoft.com/en-us/new-hope/specs/ops/docs-feature-utilization?branch=master).

## Data Storage

Azure data explorer (Kusto) is chosen now for following reasons compared to other storages:

+ Kusto engine compared with traditional SQL Server:
  + Column-oriented DBMS => OLAP-focused
  + All columns indexing automatically => Fast
+ Design for analyzing large volumes of logs or data:
  + Read, append-many
  + Delete, update almost never
+ Nearly permanent storage (10 years), where Application Insight is 90 days.
+ Supported by powerBI as data source


