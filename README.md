# Sandbox Factory for Oracle Cloud — foundation stack

Self-service OCI sandboxes that build themselves from a chat and delete themselves when their time is up.

This module is the **foundation** of [Sandbox Factory](https://github.com/ashishsinha1602/oci-sandbox-factory): one Terraform stack that installs the whole factory in your tenancy. After it applies, your users sign in to an APEX application, describe what they need in a chat (or paste an AWS-to-OCI mapping, or hand over a Git folder), and the factory plans it, prices it from Oracle's price list, builds it with Terraform on Resource Manager, and destroys it when its lifetime ends. Users need no OCI account and never see a key.

**Two-minute demo:** https://www.youtube.com/watch?v=nuHfzOqG4io

## Install with one click

[![Deploy to Oracle Cloud](https://oci-resourcemanager-plugin.plugins.oci.oraclecloud.com/latest/deploy-to-oracle-cloud.svg)](https://cloud.oracle.com/resourcemanager/stacks/create?zipUrl=https://github.com/ashishsinha1602/oci-sandbox-factory/releases/latest/download/sandbox-factory-foundation.zip)

About 15 minutes. Needs a tenancy administrator and a pay-as-you-go account. Read [PREREQUISITES](https://github.com/ashishsinha1602/oci-sandbox-factory/blob/main/docs/PREREQUISITES.md) and [SECURITY](https://github.com/ashishsinha1602/oci-sandbox-factory/blob/main/docs/SECURITY.md) first: the stack creates compartments, a VCN, a budget, IAM dynamic groups and policies, an Always Free Autonomous Database for control, and three worker container instances.

Free Tier account? Set `edition = "free"` (Always Free resources only; see FREE-TIER in the project docs).

## Use as a Terraform stack

```hcl
module "sandbox_factory" {
  source  = "ashishsinha1602/sandbox-factory/oci"
  version = "1.1.0"

  tenancy_ocid            = "ocid1.tenancy.oc1..xxxx"
  region                  = "us-phoenix-1"
  parent_compartment_ocid = "ocid1.tenancy.oc1..xxxx"   # or a compartment you administer
  budget_alert_email      = "you@example.com"
  owner                   = "you@example.com"
}
```

Authentication is not configured in the module: locally it uses your `~/.oci/config` profile, on Resource Manager it uses the resource principal. The same code runs in both places.

| Input | Required | What it is |
|---|---|---|
| `tenancy_ocid` | yes | OCID of the tenancy (root compartment) |
| `region` | yes | Region to install the factory in; IAM, budget and quotas go to the home region automatically |
| `parent_compartment_ocid` | yes | Compartment under which the sandbox tree is created |
| `budget_alert_email` | yes | Recipient of budget alerts |
| `owner` | yes | Default owner tag value |
| `prefix` | no | Short name for the compartment tree, tag namespace, VCN, budget and quotas |
| `budget_amount` | no | Monthly budget for the whole sandbox tree |
| `enable_quotas` / `quota_limits` | no | Hard service quotas on the sandbox compartment |
| `vcn_cidr`, `public_subnet_cidr`, `private_subnet_cidr`, `admin_cidr` | no | Network layout |
| `app_admin_user` / `app_admin_password` | no | First login of the application; a password is generated when left empty |

See `example.tfvars` for a complete set of values and `schema.yaml` for the Resource Manager form.

## What it builds for users

| Ask for | You get |
|---|---|
| A database to explore or to wire into agents | Autonomous Database 23ai with Select AI and REST, a chat UI, an MCP endpoint |
| Your app from a Git folder with a Dockerfile | The image built inside OCI and served on HTTPS |
| An AWS Lambda, or any small handler | An OCI Function, with Lambda code run unchanged, optionally on a Resource Scheduler cron |
| A Glue or Spark job | A Data Flow application, optionally writing Iceberg tables, plus a query application |
| An Airflow + Spark pipeline | Airflow with your DAGs loaded, Data Flow, Data Catalog, the gold tables in Oracle |
| Queues, NoSQL, Kafka, buckets | OCI Queue, NoSQL tables, a Kafka cluster, Object Storage |

Every sandbox is tagged, budgeted and owned by one person, and the reaper destroys it after 1 to 30 days.

## Licence

Apache License 2.0, see [LICENSE](LICENSE). Source: https://github.com/ashishsinha1602/oci-sandbox-factory-src
