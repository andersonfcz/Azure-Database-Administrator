# Azure Database Administrator

******************************

## Microsoct Intelligent Data Platform roles

There are five different role-based certifications offered by Microsoft focused in data.

| Role | Definition |
| ---- | ---------- |
| Data Engineer | Design and implement the management, monitoring, security and privacy of data. |
| Data Analyst | Design and build scalable data models, cleaning and transforming data, enabling advanced analytics capabilities through data visualizations. |
| Data Scientist | Applies their knowledge of data science and machine learning to implement and run machine learning workloads. |
| AI Engineer | Use Cognitive Services, machine learning, and knowledge mining to architect and implement AI solutions involving natural language processing (NLB), speech, computer vision, bots, and agents.|
| Database Administrator | Implements and manages the operational aspects of cloud-native and hybrid data platform solutions. |

## SQL Server in Azure virtual machine

A SQL Server running in an Azure virtual machine is equivalent to an on-premises SQL Server.

The reasons an application may require SQL Server running on a virtual machine include:

- ***General application support and incompatibility*** - For applications requiring and older version of SQL Server for vendo support, or application services may have a requirement to be installed with the database instance in a manner that is not compatible with a PaaS offering.
- **Use of other SQL Server Services** - In order to maximize licensing, many users choose to run SQL Server Analysis Services (SSAS), SQL Server Integration Services (SSIS), SQL Reporting Services (SSRS) on the same machine as the database engine.

### Version of SQL Server available

Microsoft keeps images of all supported versions of SQL Server available in Azure Marketplace. If you need an older version, that is covered by an extended support contract, you must install your own SQL Server binaries.

### Backup solutions

- **Backup to URL** - allows you to use standard backup syntax to back up your database to Azure Blob Storage
- **Azure Backup** - a complete enterprise backup solution that automatically handles backups across your infrastructure.

### Deployment Options

All resources in Azure share a common provider known as Azure Resource Manager (ARM) that acts as a management, and deployment service for cloud services. While there are multiple ways to deploy to Azure resources, ultimately, they all end up going into JSON documents known as Azure Resource Manager template, which is one of the deployment options for Azure resources.
The main difference between these processes is that ARM templates are a declarative approach that describes the desired structure and state of the resource, whereas the other methods can be described as imperative, which uses procedural models to explicitly specify a process to be executed.
**In large-scale deployments, the declarative approach is better and should be followed.**

### Overview of Azure storage

There are for types of storage you can use:

- Standard
- Standard SSD
- Premium SSD
- Ultra Disk

For production SQL Server data and transaction log files, you should only use Premium SSD and Ultra Disk. With premium SSD, you will see latencies in the range of 5-10 ms, with Ultra Disk you may have sub millisecond latency, but will likely see 1-2 ms workload in the real world.
You can use Standard storage for database backups, as the performance is adequate for most backup and restore workloads.

### High availability

Microsoft guarantees uptime of at least 99.99% for single instance Azure virtual machine, when using Premium SSD or Ultra Disk for all disks.

Azure offers features like availability sets, availability zones, and load-balancing to support high availability.

## Azure SQL Database for cloud-native applications

Azure SQL Database is a PaaS offer that provides high scalability capabilities and requires minimal maintenance efforts. It's aimed at new application development as it gives great deal of flexibility and granular deployment options at scale.

### Purchasing model

Comes in two main purchasing models:

#### vCore-based

Compute and storage resources are decoupled, you can scale storage and compute resources independently from one another.

| Service tier | Capability |
| ------------ | ---------- |
| General Purose | Designed for less intensive operations, offers budget oriented balanced compute and storage options. provides both provisioned and serverless compute tier. |
|Business Critical | Supports In-Memory OLTP, and buit-in read-only replica. Includes more memory per core, and uses local SSD storage, which is designed for performance-sensitive workloads. |
| Hyperscale | Introduces horizontal scaling features to add nodes as the data size grow. Only supported on single SQL database. Allows to scale storage and compute over the limits available for the General Purpose and Business Critical tiers. |

#### DTU-based

Compute and storage are dependent on the DTU level, and they provide a range of performance at a fixed storage limit, backup retention, and cost.
For example, if your database reaches the maximum storage limit, you would need to increase your DTU capacity, even if the compute utilization is low.

There are three service tiers:

- Basic
- Standard
- Premium

The scaling operation may incur in connection interruption at the end of the scaling operation. There are two main changes that trigger this behavior:

- Once you initiate a scaling operation that requires an internal failover.
- When adding or removing databases to the elastic pool.

**Azure SQL Managed Instance doesn't support DTU-based purchasing model.**

### Serverless compute tier

Can be best thought of as an autoscaling and auto-pause solution for SQL Database. It's effective for lowering the costs in development and testing environments.
The auto-pase allows you to define the period of time the database will be inactive before it is automatically paused. The auto-pause delay can be set up from 1 hour to seven days, or can be disabled.
The resume operation is triggered when the next attempt to access the database occurs. Only storage charges are applicable when the database is paused.

### Deployment Model

There are two main deployment models when deploying SQL Database:

#### Single database

The simplest deployment model, manage each of your databases individually from scale and data size perspectives. Each database has its own dedicated resources, even if deployed to the same logical server.
You can monitor resources utilization through Azure portal.

#### Elastic pool

Allows you to allocate storage and compute resources to a group of databases, rather than having to manage each database individually. Elastic pools are easier to scale than single databases, where scaling individual databases is no longer needed due to changes made to the elastic pool. You can configure resources based either on the DTU-based or vCore-based purchasing model. It is recommended to monitor your resources continually to identify concurrent performance spikes that could affect other databases part of the same elastic pool.
Is a good fit for multi-tenant architecture with low average utilization, where each tenant has its own copy of the database.

### Network options

By default has a public internet endpoint that can be controlled via firewall rules, or limited to specific Azure networks, using features like Virtual Network endpoints or Private Link.

### Backup and restore

Provides seamless backup and restore capabilities for SQL Database and SQL Managed Instance

#### Continuous backup

With SQL Database databases are backed up regularly, and are continuously copied to a read-access geo-redundant storage (RA-GRS).
Full backups are performed every week, differential backups every 12 to 24 hours, and transaction log backups every 5 to 10 minutes.

#### Geo-restore

Backups are geo-redundant by default, you can easily restore databases to a different geographical region (useful for less strict disaster recovery).

Backup storage is billed apart, the backup storage is created with the maximum size of the data tier selected for your database.

The duration of geo-restore operation can be affected by the size of the database, number of transaction logs, amount of simultaneous restore requests in the target region.

#### Point-in-time restore (PITR)

You can configure a specific point in time retention policy for each database, retention period can be set from 1 to 35 days. If not specified, the default configuration is seven days.
PITR is only supported if you are restoring a database in the same server the backup was originated.

#### Long-term retention (LTR)

Useful for scenarios that require you to set the retention policy beyound what is offered by Azure. You can set  retention policy for up to 10 years (disabled by default).

### Automatic Tuning

Buit-in feauture that relies on machine learning regression capabilities, and automatically identify tunning opportunities based on query performance.

Includes the following features:

- Identify Expensive Queries
- Forcing Last Good Execution Plan
- Adding Indexes
- Removing Indexes

To determine the best indexes for your query pattern, these indexes are tested on a copy of your database, and finally applied to your database.
All databases inherit configuration from their parent server, and can be disabled.

### Elastic query

Allows to run T-SQL queries that bridge multiple databases. Useful for applications using three and four-part names that cannot be changed. Increases portability as it allows for migration.

Support the following partitioning strategies:

- Vertical partitioning - Also called cross-database queries. The data is partitioned between many databases. Schemas are different for each database. For example, you may have a database for customer data, and a different one for payment information.
- Horizontal Partitioning - Also called sharding. The data is partitioned to distribute rows across several scaled-out databases. Schemas remain the same among all sharding databases. Support either single-tenant or multiple tenant models.

### Elastic job

Is the SQL Server Agent replacement for Azure SQL Database. To some extent, is equivalent to the Multi Server Administration feature available on an on-premises SQL Server.
You can execute T-SQL commands across sereveral targes deployments like Databases, elastic pools, shard maps. Resources can run on different subscriptions, and/or regions. The execution runs in parallel (useful when automating database maintenance tasks).

**Azure SQL Managed Instance doesn't support elastic queries and elastic job**


### SQL Data Sync

Allows you to incrementally synchronize data across multiple databases running on SQL Database or on-premises SQL Server. Good option when you need to offload intensive workloads in production with a separate database than can be used for analytics and/or ad hoc operations.

Is based on a hub topology, where you define one of the databases in the sync group to work as a hub database. The sync group can have multiple members, and you can only synchronize changes between the hub database and individual databases. Data Sync tracks changes using insert, update, and delete triggers through a historical table created on the user database.
When you create a sync group, it will ask you to provide a database responsible to store the sync group metadata. The metadata location can be either a new database or an existing one, as long in resides in the same region as your sync group.

**SQL Managed Instance doesn't support Data Sync**

## Azure SQL Database Managed Instance

Most of the features available on Azure SQL Database will also work for Azure SQL Managed Instance as they share the same base code.

Unlike Azure SQL Database, which is designed around single database, SQL Managed Instance provides several features including cross-database queries, common language runtime (CLR), access to the system database, use of the SQL Agent features.

### Hybrid licensing options

Microsoft offers benefits to SQL Server licenses, for both SQL Database and SQL Managed Instance, taking advantage of your existing licenses can reduce the cosf of running the PaaS offering.

- For each core of Enterprise Edition with Active Software Assurance, you're eligible for one vCore of SQL Database or SQL Managed Instance Business Critical, and eight vCores of General Purpose
- For each core of Standard Edition with Active Software Assurance, you're eligible for one vCore of General Purpose.

This model can reducte the total license costs by up to 40%. Effectively, you'll only be paying for the compute and storage costs, and not the software licensing costs.

### Connectivity architecture

Connections are made through TDS endpoints. While the routing and security on these connections differ, there's a gateway component that handles and routes connections to the database.

### Backup and restore

SQL Managed Instance allows easy migration of existing applications, enabling restores from on-premises backups.

There are some important considerations when running backups and restore operations:

- It isn't possible to overwrite an existing database during the restore process. Before restoring, you must ensure the database doesn't exist.
- Backups can only be restored to another managed instance. It isn't possible to restore a managed instance database backup to a SQL Server running on a virtual machine or SQL Database.
- Copy-only backup to Azure blob storage is available for SQL Managed Instance. SQL Database doesn't support this. feature.

### High availability architecture

The auto-failover groups feature allows to fail over a group of replicated databases on a server to another region. This feature is designed on top of the existing active geo-replication capability.
A failover group can include one or multiple databases, often used by the same application. Additionally, you can use the readable secondary databases to offload read-only query workloads.

**Auto-failover group is supported on both SQL Managed Instance and SQL Database**

### Migration options

There are a couple of ways to migrate on-premises databases:

- Restoring a backup - will incur more downtime, as it isn't possible to restore with **NORECOVERY** option, and apply log backups.
- Using Database Migration Service (DMS) - managed service that connects your on-premise (or Azure Virtuar Machines) SQL Server to SQL Managed Instance with near zero downtime. It acts like an automated log shipping process keeping your target databases in sync right up to the point of cutover.

### Machine Learning Services

Provides machine Learning operations within your relational database structure. Supports Python and R packages, for high-intensive predictive capabilities. Available on SQL Managed Instance, SQL Servor on VM and on-premises SQL Server.

Applications can use database on Azure combined with machine learning high-performance capabilities, where you can:

- Train machine learning models based on either sampled dataset or population dataset.
- Reduce completity in security and compliance, where you don't need to relocate your data to build and train your machine learning models.
- Deploy machine learning models using T-SQL stored procedures that support Python and R programming language.
- Use of open-source libraries like scikit-learn, PyTorch, and TensorFlow.

For busy environments, you can use T-SQL PREDICT function to accelerate predictions based on your stored model.

The Machine Learning Service can be enabled by running:

```
EXEC sp_configure 'external scripts enabled', 1;
RECONFIGURE WITH OVERRIDE;
```

The command allows the execution of external scripts and should be enabled before you attempt to use sp_execute_external_script to execute Python or R scripts in your database.

