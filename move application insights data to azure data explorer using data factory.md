# **Continuously Move Application Insights Data Using Data Factory**

## **Background**

To export telemetry from Application Insights, there are [some alternatives](https://docs.microsoft.com/zh-cn/azure/azure-monitor/app/export-telemetry) to consider.

Data factory do not support Application Insights as a direct data source ([see the supported data stores](https://docs.microsoft.com/zh-cn/azure/data-factory/copy-activity-overview#supported-data-stores-and-formats)). But we can achieve it in two ways:

+ Setup continuous export of Application Insights, which writes log blobs in the chosen storage account, then using data factory to move data from the blobs to our target storage.

+ Using the data access REST API of Application Insights as the source (data factory supports Generic REST API as the data source).

## Prerequisites

+ An Application Insights and source of data.

+ A data factory and the base knowledge of the concepts of link service, dataset, pipeline and trigger. This article only illuminates the important part and omits the basic steps such as the process of creating link service, pipeline, etc.

+ Your target output storage supported by data factory.

## Using Continuous Export

+ Application Insights part settings: [Setup Continuous Export](https://docs.microsoft.com/zh-cn/azure/azure-monitor/app/export-telemetry). Note that the blobs files are based on time partitioned file name.

+ Trigger settings: [Incrementally copy new files based on time partitioned file name by using the Copy Data tool of the data factory](https://docs.microsoft.com/zh-cn/azure/data-factory/tutorial-incremental-copy-partitioned-file-name-copy-data-tool). Set the trigger type as **Tumbling windows** and the recurrence as **every 1 hour** (the minimum granularity of the exporting time partitioned folder is hour).

+ Data source settings: write the dynamic source folder path as **/{year}/{month}/{day}/{hour}/** (change this to the exact format of the blob file names).

+ Data output settings: mapping the source data to your target format of your output storage (see [Application Insights Export Data Model](https://docs.microsoft.com/zh-cn/azure/azure-monitor/app/export-data-model)).

## Using REST API

Application Insights use Data Explorer (Kusto) to store data (traces, metrics, etc.). We can write Kusto language query in the REST API to get the desired data and also perform advanced analytics ([API guidance](https://dev.applicationinsights.io/)).

+ Trigger settings: still use **Tumbling windows**, but in this case, we can customize the frequency of data moving (the highest frequency of tumbling windows support is every 15 minutes).

+ Data source settings: to continuous export the data without overlapping, we can use **dynamic url** in the REST linked service or REST dataset part. The url can be consisted of Kusto query (e.g., where timestamp >= *start time* and timestamp < *end time*) and the parameters (*start time* and *end time*) passed from the tumbling windows (the parameters are passed from the trigger to the pipeline, then from the pipeline to the link service or dataset, [an example](https://azure.microsoft.com/mediahandler/files/resourcefiles/azure-data-factory-passing-parameters/Azure%20data%20Factory-Whitepaper-PassingParameters.pdf)).

+ Data output settings: mapping the source data to your target format of your output storage.

