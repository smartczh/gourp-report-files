# Report On Azure Data Explorer (ADX)

## Overview

+ Fast
  + Column-oriented DBMS
  + All columns indexing automaticallys
+ Support Structured, Semi-Structured and Unstructured data (everything in one database)
+ Design for large volumes of logs and data OLAP from streaming sources (build-in support ingesting from Event Hub)
  + Read, append-many supported
  + Delete, update restricted

## Representative data type

+ Dynamic: behaves somewhat like a JSON data type
  + A value of any of the primitive Kusto data types: bool, datetime, guid, int, long, real, string, and timespan.
  + An array of dynamic values: ```mv-expand``` to expand
  + Property bags (keys are unordered): ```.``` or ```[]``` to access, ```mv-expand``` to expand

## Powerful query

``` Kusto query example
StormEvents
| where StartTime >= datetime(2007-11-01) and StartTime < datetime(2007-12-01)
| where State == "FLORIDA"  
| count
```

+ Tabular data flow by ```|```
+ Performance (Especially in function, query directed task)
  + Time filters first, filter on table column rather than extend column
  + Suitable data flow
  + Prefer specific columns rather than using ```*```
  + Materializing commonly used bags of dynamic type at ingestion time
  + ...

## Update policy

Update policy is a policy set on a table that instructs Kusto to automatically append data to the table whenever new data is inserted into another table (called the source table) by running the update policy's query on the data being inserted into the source table. This allows, for example, the creation of a one table as the filtered view of another table, possibly with a different schema, retention policy, etc.

## Ingest API

SDK, REST, command

+ Stream ingest
+ Queue ingest
