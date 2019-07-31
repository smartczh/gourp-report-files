# Report On Azure Data Factory (ADF)

## Power of ADF

+ ETL (extract-transform-load) and data integration at scale
+ Support complex hybrid data source
+ Code free and easy configuration
+ High modularity and clear data flow view
+ ...

## Data-driven workflow

![ADF workflow](./Materials/ADF-workflow.png)

1. Connect&Collect: multiple accessors
2. Transform&Enrich: compute services such as HDInsight Hadoop, Spark, Data Lake Analytics, and Machine Learning
3. Publish: deployment and act
4. Monitor: built-in support, monitor via Azure Monitor, API, PowerShell, Azure Monitor logs, and health panels on the Azure portal.

## Basic concepts

+ Linked services:
  + Define the connection to the data source

``` An azure data explorer linked service example
{
    "name": "AdeLinkedService",
    "type": "Microsoft.DataFactory/factories/linkedservices",
    "properties": {
        "type": "AzureDataExplorer",
        "typeProperties": {
            "endpoint": "https://featureutilization.eastus.kusto.windows.net",
            "servicePrincipalId": "***",
            "database": "featureUtilization",
            "tenant": "**",
            "encryptedCredential": "***"
        },
        "annotations": []
    }
}
```

+ Datasets:
  + Represent data structures
  + Reference the data in the inputs or outputs

``` An azure data explorer dataset example
{
    "name": "AdeDataset",
    "properties": {
        "linkedServiceName": {
            "referenceName": "AdeLinkedService",
            "type": "LinkedServiceReference"
        },
        "annotations": [],
        "type": "AzureDataExplorerTable",
        "structure": [
            {
                "name": "name",
                "type": "String"
            },
            {
                "name": "value",
                "type": "Double"
            },
            {
                "name": "timestamp",
                "type": "DateTime"
            },
            {
                "name": "customDimensions",
                "type": "String"
            }
        ],
        "typeProperties": {
            "table": "docfxLog"
        }
    },
    "type": "Microsoft.DataFactory/factories/datasets"
}
```

+ Pipeline:
  + A logical grouping of activities that performs a unit of work
  + Custom control flow: sequence, branching, for-each, etc.

![pipeline-portal-view](./Materials/pipeline-portal-view.png)

+ Triggers: determines when a pipeline execution needs to be kicked off
  + Manual run
  + Schedule trigger
  + Tumbling window trigger
  + Event-based trigger

> Thinking: Pull and push mode in ADF  
Pull can be easily implemented by schedule or tumbling window trigger.  
Push is supported partially by event-based trigger. Now it only supports blob-create or blob-deletion event. For some data sources (Application Insights, event hub), push mode can be implemented by "event-based pull from blog logs" (Note: blog writing maybe be also delayed or scheduled).

+ Parameters:
  + Key-value pairs configuration
  + Assigned in the run context
  + Can be passed

See [an example](./Copy-Application-Insights-Data.md) (copy Application Insights data).

