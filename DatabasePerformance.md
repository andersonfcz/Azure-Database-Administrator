# Performance Monitoring

## Performance monitoring tools

The metrics that you can monitor will vary depending on the type of Azure resource you are monitoring. Azure SQL Database and SQL Server on VM will have different metrics available in the Azure portal.

When you deploy an Azure VM from the Marketplace, an agent is installed in the machine that provides a basic set of operating system metrics that are presented in Azure portal. This agent supplies metrics to a service called Azure monitor which is a platform monitoring solution that collects and displays a standard set of metricts from resources. In the case of VMs the default metrics are CPU, network utilization and disk read/write operations. You can capute additional metricts by enabling Monitoring Insights for your VM. 

These metrics pertain to the OS, not SQL Server. You are unable to view SQL Server-specific metrics from within the portal. For SQL Server-specific metrics you will need to gather them from the VM itself.

Azure Monitoring Insights allows to collect additional data points, like storage latency, available memory, and disk capacity.  This data is stored in an Azure Log Analytics workspace. Azure Log Analytics is the primary tool for storing and querying log file of all kinds in Azure. It's queried using KQL.

If you create a VM with pre-configured SQL Server image the SQL virtuar machine resource provider, which export SQL Server-specific informations like SQL data size, SQL log size, disks capacity, and manage features like automated patching and storage configuration.
This informations can been seen from the Azure portal, going to the VM -> Settings -> SQL Server Configuration -> Manage SQL Virtual Machine.

For machines created manually from outside Marketplace you need to register SQL IaaS Agent extension.

### Performance Monitor with SQL Server on an Azure VM

Windows Server has a native tool called Performance Monitor (commonly shortened to perfmon) that allows to monitor performance metrics. Perfmon operates with counters for both the OS and installed programs. When SQL Server is installed, the database engine creates its own group of specific counters. Perfmon data can be stored locally and analyzed, but you can forward results into Azure Monitor.

## Critical performance metrics

Azure Monitor allows you to to track metrics and trigger alerts or execute automated error responses.

Azure Monitor Metrics allows to not only analyze and visualize performance data, but to also trigger alerts to notify administrators or automated actions that can trigger Azure Automation runbook or a webhook. Azure Metricts data is only stored for 93 days, but you have the option to archive metrics data to Azure Storage.

Azure Monitor Alerts can be scoped in three ways.

- List of resources in one region within a subscription
- All resources in one region in one or more resource groups within a subscription
- All resources in one subscription

To create an alert:

1. In the Azure portal look for Monitor
2. Select Alerts
3. Click on Create new alert rule
4. Select the scope
5. Select the conditions
6. Select the Actions

The alerts can be configured in a static manner (when CPU goes over 95%) or in a dynamic manner using Dynamic Threshoulds. Dynamic Threshoulds learn the historical behavior of the metric and raise an alert when the resoures are operating in an abnormal manner. Dynamic Threshoulds can detect seasonality in workloads and adjust the alerting accordingly.

If Static alerts are used, you must provide a threshould for the selected metrict, for example 80% of CPU utilization.

Both types offor booleans operators such as greater than or less than, and aggegated measurements such as average, minimum, maximum, count, and total.

In order to notify admininstrators or launch an automation process, an action group can be configured.

Defining an action group is optional, and if one is not configured the alert will just log the notification to storage with no further action is taken.

With an action group, there are several ways in which you can respond to the alert.

- Automation Runbook
- Azure Function
- Email Azure Resource Manager Role
- Email/SMS/Push/Voice
- ITSM
- Azure Logic App
- Secure Webhook
- Webhook

There are two categories to these actions - notifications and automations.

### SQL Server metrics

Microsoft SQL Server is well instrumented and collects a great deal of performance metadata. The database engine has metrics that can be monitored to help identify and improve performance-related issues. Some OS metrics are only viewable from perfmom while others can be accessed through T-SQL queries, in particular, by selecting from the dynamic management views (DMVs). There are some metrics that are exposed in both locations. One example of data that can only be captured from DMVs is data and transaction log file read/write latency that is exposed in ```sys.dm_os_volume_stats```. One example of OS metric that is not available through T-SQL is the seconds per disk read and write. Combining these two metrics can help to better understand if a performance issue is related to database structure or physical storage bottleneck.

### Baseline metrics

A baseline is a collection of data measurements that helps to understand the normal "steady state" of your application or server's performance. Having the data collected over time allows to identify changes from the normal state. Baselines can be as simple as a chart of CPU utilization over time, or complex aggregations of metrics. A baseline will help to identify if an ongoing issue should be considered within normal parameters or has exceeded given threshoulds. Without a baseline, every issue could be considered nomal and not require any additional intervention.

### Correlating SQL Server and OS performance

When deploying SQL Server on an Azure VM, it's critical to correlate the performance of SQL Server with the OS. If you are using Linux as the OS, you will need to install InfluxDB, Collectd, and Grafana to capture data similar to Windows Perfmom. These services collect data from SQL Server and provide a graphical interface to review the data. These tools can be used in conjunction to look at SQL Server-specific data such as SQL Server wait statistics. Using these tools together will allow to identify bottlenecks in hardware or cade. 

The following are Performance counters of useful Windows metrics.

- Processor(_Total)%Processor Time - Measures the CPU utilization of all of the proccesors on the server. It's a good idication of the overall workload, and when used with other counters, can identify problems with query performance.
- Paging File(_Total)% Usage - In a properly configured SQL Server, memory should not page to the paging file on disk. However, in some configurations you may have other services running that consume system memory and lead to the OS paging memory to disk resulting in performance degradation
- PhysicalDisk(_Total)\Avg. Disk sec/Read and Avg Disk sec/Write - This counte provides a good metric for how the storage subsystem is working. Your latency values in most cases should not be above 20ms, and with Premium Storage you should see values less than 10ms.
- System\Processor Queue Length - Indicates the number of threads that are waiting for the time on the processor. If it's greater than zero, it indicates CPU pressure, indicating your workload needs more CPUs.
- SQLServer:Buffer Manager\Page life expectancy - Page life expectancy indicates how long SQL Server expects a page to live in memory. There is no proper value for this setting. Older documentation refers to 300 seconds as proper, but what was written in a 32-bit era when servers had far less RAM. You should monitor this value over time, and evaluate sudden drops. Such drops in the counter's value could indicate poor query patterns, external memory pressure (server running in a large SSIS package) or could just be normal system processing like running a consistency check on a large database.
- SQLServer:SQL Statistics\Batch Requests/sec - This counter is good for evaluating how consistently busy a SQL Server is over time. There is no good or bad value, but you can use in conjunction with % Processor time to better understand your workload and baselines.
- SQLServer:SQL Statistics\SQL Compilations/sec and SQL Re-Compilations/sec - These counters will be updated when SQL Server has to compile or recompile execution plan for a query because there is no existing plan in the cache, or because a plan was invalidated because of a change. Recompiles can indicate T-SQL with recompile query hints, or be indicative of memory pressure on the plan cache caused by either many ad-hoc queries or simple memory pressure

### Wait statistics

When a thread is being executed and is forced to wait on an unavailable resource, SQL Server keeps track of these metrics. This information is identifiable via DMV ```sys.dm_os_wait_stats```. This information is important to understading the baseline performance of your database and help to identify specific performance issues with query execution and hardware limitations. Identifying the appropriate wait type and corresponding resolution will be critical for resolving performance issues.

## Extended events

The extended events engine in Azure SQL is a monitoring system that allows to capture information about activity in your databases and servers. The monitoring solutions on the Azure platform allow you to configure monitoring for you environment and provide automated responses to error conditions.

Extended events build on the functionality of SQL Server Profiler by allowing you to trace queries and by exposing additinal data (events) that you can monitor.

Some issues you can troubleshoot with Extended events:

- Troubleshooting blocking and deadlocking performance issues
- Identifying long-running queries
- Monitoring DDL operations
- Logging missing column statistics
- Observing Memory Pressure in your database
- Long-running physical I/O operations

Extended event framework allows you to use filters to limit the amount of data you collect in order to reduce the overhead of data collection, and allows you do identify you performance problem by targeting you focus onto specific areas.

Extended events cove the full surface area of SQL Server, and are divided into four channels:

- Admin - events that are targeted for end users and administrators. The events included indicate a problem within a well-defined set of actions an administrator can take. An examle is the generation of an XML deadlock report to help identify the root cause of the deadlock.
- Operational - events that are used for analysis and diagnostics or common problems. These events can be used to trigger an action or task based on an occurrence of the event. An example would be a database in an availabality group changing state, which would indicate a failover.
- Analytic - events related to performance and are published in high volume. Tracing stored procedure or query execution would be an example.
- Debug - are not necessarily fully documented and should only be used when troubleshooting in conjunction with Microsoft support.

Events are added to sessions, which can host multiple events. Typically event are grouped together in a session to capture a related set of information.

This query obtain a list of the available events, actions and targets:

```SQL
SELECT
    obj.object_type,
    pkg.name AS [package_name],
    obj.name AS [object_name],
    obj.description AS [description]
FROM sys.dm_xe_objects  AS obj
    INNER JOIN sys.dm_xe_packages AS pkg  ON pkg.guid = obj.package_guid
WHERE obj.object_type in ('action',  'event',  'target')
ORDER BY obj.object_type,
    pkg.name,
    obj.name;
```

### Creating extended events session

Basic process of creating an extended event sessiens using the New Session dialog from SSMS.

Expand Management node in SSMS

Expand the Extended Events node, right-click on Sessions and select New Session.

Name the session and select one of the templates. Templates are groupd into four categories:
   - Locks and Blocks
   - Profiler Equivalents
   - Query Execution
   - System Monitoring

Chose the options for when to start the session. You can choose to start whenever the server starts, as soon as it's been created. You also have the option for enabling the causality tracking, which adds a GUID and sequence number to the output of each event, which allow to identify the order that the events occurred.

Add Events to your session - An event represents a point of interest within the database engine code, these can represent internal system operations, or can be associated with user actions like query execution. 

By default the session would capture all instances of events taking place on your session, but you can limit the collection. 

Global fields, allow to choose the data you are collecting, when your event occurs. Global fields are also known as actions, as the action is to add additional data fields to the event. These fields represent the data that is collected when the extended event occurs, and are common across most extended events.

Filters are a powerfull feature of Extended events that allow to control the capture of only specific occurrences of the event. Filters apply to a single field on a single event.

It's a good practice to configure a filter for each event that you are capturing. These are specific to the event being triggered, and can include optional fields for collection.

An extended event session has a target - it can be simply thought of as a place for the engine to keep track of occurrences of an event. Two of the more common targets are event file, which is a file on the file system that can store events, and in SQL PaaS offering this data is written to a blob storage. Another common target is the ring buffer which is within SQL Server's memory. The ring buffer is most commonly used for live observation of an event session as it's a circular buffer and data is not persisted beyond a session. Most targets process data asynchronously, which means the event data is written to memory before being persisted to disk. The exception is Event Tracing for Windown target (ETW) and Event Counter target, which are processed synchronously.

| Target | Description | Processing |
| ------ | ----------- | ---------- |
| Event Counter | Counts all events that occurred during an Extended Event session. It's used to obtain information about workload characteristics about a workload without the overhead of a full event collection | Synchronously |
| Event file | Writes event session output from memory onto persistent file on disk | Asynchronous |
| Event Pairing | Many events that generally occur in pairs (lock acquire, lock release). Can be used to identify when those events do not occur in a matched set | Asynchronous |
| Event Tracing for Windows | Similar to event counter, which counts the occurrences of an event. The difference is that the histogram can count based on a specific event column or action | Asynchronous |
| Ring Buffer | Used to hold data in memory. Data is not persisted to disk and maybe frequently flushed from the buffer | Asynchronous |

You can create an extended event session using T-SQL:

```SQL
IF EXISTS (SELECT * FROM sys.server_event_sessions WHERE name='test_session')
    DROP EVENT session test_session ON SERVER;
GO

CREATE EVENT SESSION test_session
ON SERVER
    ADD EVENT sqlos.async_io_requested,
    ADD EVENT sqlserver.lock_acquired
    ADD TARGET package0.etw_classic_sync_target (SET default_etw_session_logfile_path = N'C:\demo\traces\sqletw.etl' )
    WITH (MAX_MEMORY=4MB, MAX_EVENT_SIZE=4MB);
GO
```

Event sessions can be scoped to a server or a database. After you create the session, you'll have to start it. You can do through T-SQL and ```ALTER```  the session using the ```STATE``` option.

## Azure SQL Insights

SQL Insights is a component that allows you to analyze your queries, and tune performance.

With SQL Insights interactive features, you can customize telemetry collection and frequency, and combine data from multiple sources into a single monitoring.

SQL Insights collects data remotely from DMVs, and it's built on top of the Azure Monitor, giving access to the native alerting and visualizations. It retains a set of metrics over time, which allows you to investigate performance issues.

To user SQL Insights you need a dedicated virtual machine that will monitor and remotely collect data from your SQL servers. This VM needs to have these components installed:

- Azure Monitor agent
- Workload Insights extension

In order to increase control over charges, you can choose which telemetry data to collect, the frequency, and retention policy parameters. Database activity and settings that you've set in your monitoring profiles will determine the amount of data being collected and the cost.

You can access performance data from the SQL Insights workbook template, or directly from the monitoring logs.

### SQL Insights in Azure Monitor

1. from the Azure portal looks for Montior and select SQL (preview), then Create new profile
2. Configure the following components:
   - Monitoring profile - group servers, instances or databases to monitor
   - Log Analytics workspace - where to send the monitoring data
   - Collection settings - customize the data collection for your profile. The default settings cover the majority of monitoring scenarios.
3. Select Create monitoring profile
4. Select Manage profile, then Add monitoring machine. At this time the only supported virtual machine OS is Ubuntu 18.04. the following pre-requisites are satisfied:
   -  Set permission for SQL accounts
   -  Create firewall and networking rules for SQL

### Limitations

SQL Insights has no support or has limited support for the following components:

- Non-Azure instances
- Azure SQL Database elastic pools
- Azure SQL Database running on Basic, S0, S1, and S2 service tiers
- Azure SQL Database serverless tier
- Multiple secondary replicas
- Authentication with Azure ACtive Directory (Only SQL authentication is supported)

SQL Insights brings together performance metricts at scale in a single view.

In addition to visualization and data collection, it has buit-in intelligence for troubleshooting activities. It allows for custom monitoring alerts and rules that allow for quick identification and resolution of issues.

## Query Performance Insight

Identifying which queries are consuming the most resources is the first step in any database performance tuning.

Query Performance Insight allows to identify expensive queries.

Query Performance Insights has three buttons to allow to filter:

- long running queries
- top resource consuming queries
- custom filter

The default vaule is Resource Consuming Queries. This tab shows the top five queries sorted by the particular resource that you select.

The Long Running Queries, the metrics are limited to the top five queries sorted by duration from the previous 24 hours and is a sum aggregation.

In the Custom tab, there's a little more flexibility. It offers sereval metrics, CPU, Log IO, Data IO, and memory.

You can drill into an individual query and see the query ID and the query itself, as well as the query aggregation type and associated time period. The query ID correlates to the query ID located within the Query Store. Metrics from Query Performance Insight can be located within the Query Store for deeper analysis or possibly problem resolution.

Query Performane Insight doesn't show the query's execution plan, but you can quickly identify that query and use the information to extract the plan from the Query Store.
