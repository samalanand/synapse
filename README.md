# Azure Synapse Workloads and Service Levels - Architects guide

Azure Synapse Analytics is a powerful enterprise analytics service that brings together the best of SQL technologies used in enterprise data warehousing to the distributed processing with big data and data lake. Dedicated SQL pool (formerly SQL DW) refers to the enterprise data warehousing features that are available in Azure Synapse Analytics.

While it fits the needs of a modern data warehouse, it gives a challenge to the architects in determining the compute and processing requirements (referred to as Service levels of the sql pool - explained later) for the Dedicated SQL pool. These concerns can be frustrating in a cloud environment where resources should be horizontally scalable and elastic.

This is not to say that a Dedicated poll cannot be moved between Service levels - it absolutely can, but you need to be prepared for cost implications for every increase ranging between $1 per hour to $360 per hr. This equates to a huge difference of $1000 to $400,000 for a pool running a month and results in very unpredictable costs for an organization.

This article brings together the relevant information from Microsoft documentation into a toolkit for you as a Data Architect to assist with during Azure Synapse Workload management decision making. There are two components to this toolkit.
1.  Service level matrix - provides a combined view of all performance metrics for each service level
2.  Workload visualizer - visual simulation to help find the service level for the workload requirements



## 1. Service level matrix

Service levels are based on Synapse's Data Warehousing Units (DWU); which represents a combination of CPU, memory, and IO bundled into units of compute scale.

The image below has the combined view of all Synapse dedicated pool service levels and its corresponding metrics.
  
Dedicated SQL pool Service levels![No alt text provided for this image](https://media-exp1.licdn.com/dms/image/C4E12AQENuexqdU0cig/article-inline_image-shrink_1000_1488/0/1633380578708?e=1639008000&v=beta&t=n6FgUaOYLXtlt1poxuZi1Ku6s6LT2QGLJ9nFcV5uIgM)

-   Service levels are represented as DWUs. Increasing the service level means, increasing the number of DWUs which in turn increases the maximum number of concurrent queries and concurrency slots. A DWU1000c service level offers 1000 DWUs with 40 concurrency slots which can support a maximum of 32 concurrent processes.
-   Concurrency slots are conceptual in that they provide a convenient way to understand resource usage and availability for workload and query execution. You can think of them as cups available on a cup cake baking tray because there's only limited pre defined number available based on the service level. The higher the service level, the larger the baking tray, and the more cupcakes (concurrent processes) can be created simultaneously.
-   Resource classes are pre-determined resource limits in Synapse dedicated SQL pool that govern compute resources allocated for workloads. There are two types of resource classes . Static Resource Classes (staticrc10, staticrc20 etc.) allocate the same amount of memory regardless of the service level. Dynamic Resource Classes (smallrc, mediumrc etc.) allocate a variable amount of memory depending on the service level.
-   The Service level matrix above represents concurrency slots used by various resource classes across service levels. Every workload and related query will have an associated resource class assigned to it (the default allocation is smallrc). A query acquires concurrency slots before execution and releases after it completes.
-   Based on the resource class assigned to a workload, the same query can be run with different number of concurrency slots with varying performance (a query running with 10 slots can access 5 times more compute resources than a query running with 2 concurrency slots). If all workloads use same resource class with 10 concurrency slots per query and if there are total of 40 concurrency slots available, then only 4 queries can run concurrently.
-   Until DW1000c it uses a single node cluster, so the real benefits of MPP (Massively parallel processing) architecture with compute and storage across multiple nodes are not effectively utilized.
-   A maximum of 128 concurrent queries will execute and remaining queries will be queued.
-   TempDB is based on DWUs at 399 GB per DW100c. Mean a DWU1000c instance will get 3.99 TB tempdb.
-   The pricing details for each service levels are intentionally excluded due to possibility of this changing more often and by the region. You can get this from Azure page (https://azure.microsoft.com/en-us/pricing/details/synapse-analytics/)


## 2. Workload visualizer

Now that we know which service level has what capacity how do we know which one you should plan for? There is no simple answer for this. It is based on the kind of workloads the data warehouse are planned to be performing. The workload refers to all operations in relation to a data warehouse, like loading data into warehouse (dataflows, Polybase or other ETL), performing analysis, ad-hoc querying and reporting, exporting data out of warehouse etc.

Different workloads require different compute capacity at varying intervals. While there are many ways to classify data warehousing workloads, you can classify each of these query workloads with different resource classes. Eg. A data warehousing solution will often have a workload policy for load activity, such as assigning a higher resource class with more resources. A different workload policy could apply to queries, such as lower importance compared to load activities.

-   Smaller resource classes reduce the compute usage with increased concurrency.
-   Larger resource classes allocates more compute power per query, but reduce concurrency.

When you identify the workloads and your intended resource classes, you can use the Workload visualizer to identify the recommended service level. Download the spreadsheet using the link at the bottom of this post.

Use the input section of the workload visualizer to key in the number of concurrent queries against each resource class (See below).

Input ![No alt text provided for this image](https://media-exp1.licdn.com/dms/image/C4E12AQFJkU4rNIMQfQ/article-inline_image-shrink_1000_1488/0/1633371898568?e=1639008000&v=beta&t=sz3F66va_zFNo9ey0Lpe37QBVuPURqjxjWdPpZJA1bs)

As you key in the compute needs you can visualize the concurrency slots being filled for each service level. This sheet will help you plan for deciding the Service level of your Synapse dedicated pool instance.All your files and folders are presented as a tree in the file explorer. You can switch from one to another by clicking a file in the tree.
  
Output ![No alt text provided for this image](https://media-exp1.licdn.com/dms/image/C4E12AQHgyaq1ySdTtA/article-inline_image-shrink_1000_1488/0/1633380406449?e=1639008000&v=beta&t=nO0wELzZwiVES5aLVPXsUf7e7lhom1707wAIyUN25VU)

-   Slots available - represents total number of concurrency slots available for that service level
-   Slots used - represents the estimated number of concurrency slots used for the keyed in combination of workload requirements

If 100% of the slots are utilized the queries will go on to a queue waiting for the executing queries to complete.

Add alt text![No alt text provided for this image](https://media-exp1.licdn.com/dms/image/C4E12AQF8HW_T0yDT3w/article-inline_image-shrink_1000_1488/0/1633380735377?e=1639008000&v=beta&t=LtfqDX1GjLmi06GePdukr6Xgm1aCk-Bskw-mpz0IBmU)

Its always better to plan for 70 - 80% of the usage so that your data engineers have enough slots for ad-hoc queries. When multiple queries are fired against Synapse dedicated pool, it puts queries into a queue based on importance (classification) and concurrency slots. Queries wait in the queue until enough concurrency slots are available. Importance and concurrency slots determine CPU prioritization. More information on this will be shared later.

> **Conclusion:** If you are planning to use Azure Synapse dedicated pool, you should perform a workload analysis before you come up with your infrastructure sizing and TCO calculation. You should be ready with two or three service level options and then start with the lowest one and observe the performance.
