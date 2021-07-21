# CDP Private Cloud Base

CDP Private Cloud Base is an on-premises version of Cloudera Data Platform.

This new product combines the best of Cloudera Enterprise Data Hub and Hortonworks Data Platform Enterprise along with new features and enhancements across the stack. This unified distribution is a scalable and customizable platform where you can securely run many types of workloads.

Figure 1. CDP Private Cloud Base Architecture

![](https://docs.cloudera.com/cdp-private-cloud/latest/overview/images/cdppvc-architecture.png)

CDP Private Cloud Base supports a variety of hybrid solutions where compute tasks are separated from data storage and where data can be accessed from remote clusters, including workloads created using CDP Private Cloud Experiences. This hybrid approach provides a foundation for containerized applications by managing storage, table schema, authentication, authorization and governance.

CDP Private Cloud Base is comprised of a variety of components such as Apache HDFS, Apache Hive 3, Apache HBase, and Apache Impala, along with many other components for specialized workloads. You can select any combination of these services to create clusters that address your business requirements and workloads. Several pre-configured packages of services are also available for common workloads. These include:

## Regular (Base) Clusters

Data Engineering

Process develop, and serve predictive models.

Services included: HDFS, YARN, YARN Queue Manager, Ranger, Atlas, Hive, Hive on Tez, Spark, Oozie, Hue, and Data Analytics Studio

Data Mart

Browse, query, and explore your data in an interactive way.

Services included: HDFS, Ranger, Atlas, Hive, and Hue

Operational Database

Real-time insights for modern data-driven business.

Services included: HDFS, Ranger, Atlas, and HBase

Custom Services

Choose your own services. Services required by chosen services will automatically be included.

## Compute Clusters

Data Engineering

Process develop, and serve predictive models.

Services included: Spark, Oozie, Hive on Tez, Data Analytics Studio, HDFS, YARN, and YARN Queue Manager

Spark

Spark for Compute

Services included: Core Configuration, Spark, Oozie, YARN, and YARN Queue Manager

Data Mart

Impala for Compute

Services included: Core Configuration, Impala, and Hue

Streams Messaging (Simple)

Simple Kafka cluster for streams messaging

Services included: Kafka, Schema Registry, and Zookeeper

Streams Messaging (Full)

Advanced Kafka cluster with monitoring and replication services for streams messaging

Services included: Kafka, Schema Registry, Streams Messaging Manager, Streams Replication Manager, Cruise Control, and Zookeeper

Custom Services

Choose your own services. Services required by chosen services will automatically be included.

When installing a CDP Private Cloud Base cluster, you install a single parcel called Cloudera Runtime that contains all of the components. For a complete list of the included components, see [Cloudera Runtime Component Versions](https://docs.cloudera.com/cdp-private-cloud-base/7.1.6/runtime-release-notes/topics/rt-pvc-runtime-component-versions.html).

In addition to the Cloudera Runtime components, CDP Private Cloud Base includes powerful tools to help manage, govern, and secure your cluster.

## CDP Private Cloud Base Tools

### Cloudera Manager

CDP Private Cloud Base uses Cloudera Manager to manage one or more clusters and their configurations and to monitor cluster performance. You also use Cloudera Manager to manage installations, upgrades, maintenance workflows, encryption, access controls, and data replication. You can also use Cloudera Manager to manage Cloudera Enterprise CDH clusters. You can use Cloudera Manager to create a _Virtual Private cluster_ that allows you to separate compute resources from data storage and to share data storage among compute resources.

### Apache Atlas

Also included in CDP Private Cloud Base is Apache Atlas, used to provide governance for your data. Apache Atlas serves as a common metadata store that is designed to exchange metadata both inside and outside of the Hadoop stack. Close integration of Atlas with Apache Ranger enables you to define, administer, and manage security and compliance policies consistently across all components of the Hadoop stack. For customers familiar with Cloudera Enterprise, Apache Atlas replaces Cloudera Navigator Metadata Server. It provides the following capabilities:

-   Flexible metadata models
    
-   Entity search using model attributes, classifications (tags), and free text
    
-   Lineage across entities based on processes applied to the entities
    

### Apache Ranger

Apache Ranger provides auditing, authentication, and authorization functionality for your CDP Private Cloud Base clusters.

Apache Ranger provides a centralized framework for collecting access audit history and reporting data, including filtering on various parameters. Ranger enhances audit information obtained from Hadoop components and provides insights through this centralized reporting capability.

Apache Ranger also manages access control through a user interface that ensures consistent policy administration across CDP Private Cloud Base components. Security administrators can define security policies at the database, table, column, and file levels, and can administer permissions for specific LDAP-based groups or individual users. Rules based on dynamic conditions such as time or geolocation can also be added to an existing policy rule. The Ranger authorization model is pluggable and can be easily extended to any data source using a service-based definition.

For customers familiar with Cloudera Enterprise, Apache Ranger replaces Sentry and Navigator Audit Server and also provides the following capabilities:

-   Better fine-grained access controls:
    
    -   Dynamic Row Filtering
        
    -   Dynamic Column Masking
        
    -   Attribute-based Access Control
        
    -   SparkSQL fine-grained access control
        
-   Rich policy features
    
    -   Allow/Deny constructs, Custom policy conditions/context enrichers, time bound policies, Atlas integration (for tag based policies)
        
-   Extensive Access Auditing with rich event metadata