---
layout: post
title: EMR Serverless Cross-Account Access to Iceberg Tables
subtitle: Learn how to make your Iceberg tables available for spark jobs running cross-account on EMR Serverless
tags: [blog]
comments: false
---

A few weeks ago, I was working on a project where I had to access Iceberg tables from a Spark job running on EMR cluster in another account. I found it a bit tricky to set up, so I decided to write this post to help others who might be facing the same issue.

If you follow the EMR documentation on how to access Iceberg tables you're going to find the following `spark-submit` parameters recommendation:

```bash
--conf spark.jars=/usr/share/aws/iceberg/lib/iceberg-spark3-runtime.jar
--conf spark.sql.extensions=org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions
--conf spark.sql.catalog.<YOUR_CATALOG_NAME_HERE>=org.apache.iceberg.spark.SparkCatalog 
--conf spark.sql.catalog.<YOUR_CATALOG_NAME_HERE>.catalog-impl=org.apache.iceberg.aws.glue.GlueCatalog 
--conf spark.sql.catalog.<YOUR_CATALOG_NAME_HERE>.warehouse=s3://DOC-EXAMPLE-BUCKET/EXAMPLE-PREFIX/
--conf spark.hadoop.hive.metastore.client.factory.class=com.amazonaws.glue.catalog.metastore.AWSGlueDataCatalogHiveClientFactory
```

After setting those and configuring the cross-account access on your AWS Glue Catalog in the account where the iceberg table lives, you're going to receive an error similar to `org.apache.hadoop.hive.ql.metadata.HiveException: Unable to fetch table table_name. StorageDescriptor#InputFormat cannot be null for table: table_name(Service: null; Status Code: 0; Error Code: null; Request ID: null; Proxy: null)`.

Considering you've done everything right while setting the permissions you can solve this issue adding the following parameter:

```bash
--conf spark.sql.catalog.<YOUR_CATALOG_NAME_HERE>.glue.id=<ICEBERG_TABLE_ACCOUNT_ID>
```

After this, you can access the database using SparkSQL:

```sql
SELECT *
FROM <YOUR_CATALOG_NAME_HERE>.<DATABASE>.<TABLE_NAME>
```

That's all for this post, hope it helps!
