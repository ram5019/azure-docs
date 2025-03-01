---
title: Management operations in Azure Managed Instance for Apache Cassandra
description: Learn about the management operations supported by Azure Managed Instance for Apache Cassandra. It also explains separation of responsibilities between the Azure support team and customers when maintaining standalone and hybrid clusters.
author: TheovanKraay
ms.author: thvankra
ms.service: managed-instance-apache-cassandra
ms.topic: overview
ms.date: 11/02/2021
ms.custom: ignite-fall-2021
---

# Management operations in Azure Managed Instance for Apache Cassandra

Azure Managed Instance for Apache Cassandra provides automated deployment and scaling operations for managed open-source Apache Cassandra data centers. This article defines the management operations and features provided by the service. It also explains the separation of responsibilities between the Azure support team and customers when maintaining standalone and [hybrid](configure-hybrid-cluster.md) clusters.

## Compaction

* The system currently doesn't perform a major compaction.
* Repair (see [Maintenance](#maintenance)) performs a Merkle tree compaction, which is a special kind of compaction.
* Depending on the compaction strategy on the keyspace, Cassandra automatically compacts when the keyspace reaches a specific size. We recommend that you carefully select a compaction strategy for your workload, and don't do any manual compactions outside the strategy.

## Patching

* Operating System-level patches are done automatically at approximately 2-week cadence.

* Apache Cassandra software-level patches are done when security vulnerabilities are identified. The patching cadence may vary.

* During patching, machines are rebooted one rack at a time. You shouldn't experience any degradation at the application side as long as **quorum ALL setting is not being used**, and the replication factor is **3 or higher**.

* The version in Apache Cassandra is in the format `X.Y.Z`. You can control the deployment of major (X) and minor (Y) versions manually via service tools. Whereas the Cassandra patches (Z) that may be required for that major/minor version combination are done automatically.

>[!NOTE]
> The service currently supports Cassandra versions 3.11 and 4.0. By default, version 3.11 is deployed, as version 4.0 is currently in public preview. See our [Azure CLI Quickstart](create-cluster-cli.md) (step 5) for specifying Cassandra version during cluster deployment.

## Maintenance

* The [Nodetool repair](https://docs.datastax.com/en/cassandra-oss/3.x/cassandra/tools/toolsRepair.html) is automatically run by the service using [reaper](http://cassandra-reaper.io/). This tool is run once every week. You may wish to disable it if using your own service for a [hybrid deployment](configure-hybrid-cluster.md).

* Node health monitoring consists of:

  * Actively monitoring each node's membership in the Cassandra ring.
  * Actively monitoring virtual machines to identify and fix problems with Azure, Virtual Machines, storage, Linux, and the support software.

## Support

Azure Managed Instance for Apache Cassandra provides an [SLA](https://azure.microsoft.com/support/legal/sla/managed-instance-apache-cassandra/v1_0/) for the availability of data centers in a managed cluster. If you encounter any issues with using the service, file a [support request](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest) in the Azure portal.

>[!IMPORTANT]
> We will attempt to investigate and diagnose any issues reported via support case, and resolve or mitigate where possible. However, you are ultimately responsible for any Apache Cassandra configuration level usage which causes CPU, disk, or network problems.
>
> Examples of such issues include:
>
>  * Inefficient query operations.
>  * Throughput that exceeds capacity.
>  * Ingesting data that exceeds storage capacity.
>  * Incorrect keyspace configuration settings.
>  * Poor data model or partition key strategy.
>
> In the event that we investigate a support case and discover that the root cause of the issue is at the Apache Cassandra configuration level (and not any underlying platform level aspects we maintain), the case may be closed. Where possible, we will also provide recommendations and guidance on remediation. We therefore recommend you [enable metrics](visualize-prometheus-grafana.md) and/or become familiar with our [Azure monitor integration](monitor-clusters.md ) in order to prevent common application/configuration level issues in Apache Cassandra, such as the above.

>[!WARNING]
> Azure Managed Instance for Apache Cassandra also let's you run `nodetool` and `sstable` commands for routine DBA administration - see article [here](dba-commands.md). Some of these commands can destabilize the cassandra cluster and should only be run carefully and after being tested in non-production environments. Where possible, a `--dry-run` option should be deployed first. Microsoft cannot offer any SLA or support on issues with running commands which alter the default database configuration and/or tables.

## Backup and restore

Snapshot backups are enabled by default and taken every 4 hours with [Medusa](https://github.com/thelastpickle/cassandra-medusa). Backups are stored in an internal Azure Blob Storage account and are retained for up to 2 days (48 hours). There's no cost for backups. To restore from a backup, file a [support request](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest) in the Azure portal.

> [!WARNING]
> Backups can be restored to the same VNet/subnet as your existing cluster, but they cannot be restored to the *same cluster*. Backups can only be restored to **new clusters**. Backups are intended for accidental deletion scenarios, and are not geo-redundant. They are therefore not recommended for use as a disaster recovery (DR) strategy in case of a total regional outage. To safeguard against region-wide outages, we recommend a multi-region deployment. Take a look at our [quickstart for multi-region deployments](create-multi-region-cluster.md).

## Security

Azure Managed Instance for Apache Cassandra provides many built-in explicit security controls and features:

* Hardened Linux Virtual Machine images with a controlled supply chain.
* Common Vulnerability & Exposure (CVE) monitoring at the Operating System level.
* Certificate rotation for both Apache Cassandra and Prometheus software hosted on the managed Virtual Machines.
* Active vulnerability scanning.
* Active virus scanning.
* Secure coding practices.

For more information on security features, see our article [here](security.md).

## Hybrid support

When a [hybrid](configure-hybrid-cluster.md) cluster is configured, automated reaper operations running in the service will benefit the whole cluster. This includes data centers that aren't provisioned by the service. Outside this, it is your responsibility to maintain your on-premises or externally hosted data center.

## Next steps

Get started with one of our quickstarts:
* [Create a managed instance cluster from the Azure portal](create-cluster-portal.md)
* [Deploy a Managed Apache Spark Cluster with Azure Databricks](deploy-cluster-databricks.md)
* [Manage Azure Managed Instance for Apache Cassandra resources using Azure CLI](manage-resources-cli.md)
