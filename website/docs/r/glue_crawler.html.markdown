---
subcategory: "Glue"
layout: "aws"
page_title: "AWS: aws_glue_crawler"
description: |-
  Manages a Glue Crawler
---

# Resource: aws_glue_crawler

Manages a Glue Crawler. More information can be found in the [AWS Glue Developer Guide](https://docs.aws.amazon.com/glue/latest/dg/add-crawler.html)

## Example Usage

### DynamoDB Target

```hcl
resource "aws_glue_crawler" "example" {
  database_name = aws_glue_catalog_database.example.name
  name          = "example"
  role          = aws_iam_role.example.arn

  dynamodb_target {
    path = "table-name"
  }
}
```

### JDBC Target

```hcl
resource "aws_glue_crawler" "example" {
  database_name = aws_glue_catalog_database.example.name
  name          = "example"
  role          = aws_iam_role.example.arn

  jdbc_target {
    connection_name = aws_glue_connection.example.name
    path            = "database-name/%"
  }
}
```

### S3 Target

```hcl
resource "aws_glue_crawler" "example" {
  database_name = aws_glue_catalog_database.example.name
  name          = "example"
  role          = aws_iam_role.example.arn

  s3_target {
    path = "s3://${aws_s3_bucket.example.bucket}"
  }
}
```


### Catalog Target

```hcl
resource "aws_glue_crawler" "example" {
  database_name = aws_glue_catalog_database.example.name
  name          = "example"
  role          = aws_iam_role.example.arn

  catalog_target {
    database_name = aws_glue_catalog_database.example.name
    tables        = [aws_glue_catalog_table.example.name]
  }

  schema_change_policy {
    delete_behavior = "LOG"
  }

  configuration = <<EOF
{
  "Version":1.0,
  "Grouping": {
    "TableGroupingPolicy": "CombineCompatibleSchemas"
  }
}
EOF
}
```

### MongoDB Target

```hcl
resource "aws_glue_crawler" "example" {
  database_name = aws_glue_catalog_database.example.name
  name          = "example"
  role          = aws_iam_role.example.arn

  mongodb_target {
    connection_name = aws_glue_connection.example.name
    path            = "database-name/%"
  }
}
```

## Argument Reference

~> **NOTE:** Must specify at least one of `dynamodb_target`, `jdbc_target`, `s3_target` or `catalog_target`.

The following arguments are supported:

* `database_name` (Required) Glue database where results are written.
* `name` (Required) Name of the crawler.
* `role` (Required) The IAM role friendly name (including path without leading slash), or ARN of an IAM role, used by the crawler to access other resources.
* `classifiers` (Optional) List of custom classifiers. By default, all AWS classifiers are included in a crawl, but these custom classifiers always override the default classifiers for a given classification.
* `configuration` (Optional) JSON string of configuration information.
* `description` (Optional) Description of the crawler.
* `dynamodb_target` (Optional) List of nested DynamoDB target arguments. See below.
* `jdbc_target` (Optional) List of nested JBDC target arguments. See below.
* `s3_target` (Optional) List nested Amazon S3 target arguments. See below.
* `mongodb_target` (Optional) List nested MongoDB target arguments. See below.
* `schedule` (Optional) A cron expression used to specify the schedule. For more information, see [Time-Based Schedules for Jobs and Crawlers](https://docs.aws.amazon.com/glue/latest/dg/monitor-data-warehouse-schedule.html). For example, to run something every day at 12:15 UTC, you would specify: `cron(15 12 * * ? *)`.
* `schema_change_policy` (Optional) Policy for the crawler's update and deletion behavior.
* `security_configuration` (Optional) The name of Security Configuration to be used by the crawler
* `table_prefix` (Optional) The table prefix used for catalog tables that are created.
* `tags` - (Optional) Key-value map of resource tags

### dynamodb_target Argument Reference

* `path` - (Required) The name of the DynamoDB table to crawl.
* `scan_all` - (Optional) Indicates whether to scan all the records, or to sample rows from the table. Scanning all the records can take a long time when the table is not a high throughput table.  defaults to `true`.
* `scan_rate` - (Optional) The percentage of the configured read capacity units to use by the AWS Glue crawler. The valid values are null or a value between 0.1 to 1.5.

### jdbc_target Argument Reference

* `connection_name` - (Required) The name of the connection to use to connect to the JDBC target.
* `path` - (Required) The path of the JDBC target.
* `exclusions` - (Optional) A list of glob patterns used to exclude from the crawl.

### s3_target Argument Reference

* `path` - (Required) The path to the Amazon S3 target.
* `connection_name` - (Optional) The name of a connection which allows crawler to access data in S3 within a VPC.
* `exclusions` - (Optional) A list of glob patterns used to exclude from the crawl.

### catalog_target Argument Reference

* `database_name` - (Required) The name of the Glue database to be synchronized.
* `tables` - (Required) A list of catalog tables to be synchronized.

### mongodb_target Argument Reference

* `connection_name` - (Required) The name of the connection to use to connect to the Amazon DocumentDB or MongoDB target.
* `path` - (Required) The path of the Amazon DocumentDB or MongoDB target (database/collection).
* `scan_all` - (Optional) Indicates whether to scan all the records, or to sample rows from the table. Scanning all the records can take a long time when the table is not a high throughput table. Default value is `true`.

~> **Note:** `deletion_behavior` of catalog target doesn't support `DEPRECATE_IN_DATABASE`.

-> **Note:** `configuration` for catalog target crawlers will have `{ ... "Grouping": { "TableGroupingPolicy": "CombineCompatibleSchemas"} }` by default.


### schema_change_policy Argument Reference

* `delete_behavior` - (Optional) The deletion behavior when the crawler finds a deleted object. Valid values: `LOG`, `DELETE_FROM_DATABASE`, or `DEPRECATE_IN_DATABASE`. Defaults to `DEPRECATE_IN_DATABASE`.
* `update_behavior` - (Optional) The update behavior when the crawler finds a changed schema. Valid values: `LOG` or `UPDATE_IN_DATABASE`. Defaults to `UPDATE_IN_DATABASE`.

## Attributes Reference

In addition to all arguments above, the following attributes are exported:

* `id` - Crawler name
* `arn` - The ARN of the crawler

## Import

Glue Crawlers can be imported using `name`, e.g.

```
$ terraform import aws_glue_crawler.MyJob MyJob
```
