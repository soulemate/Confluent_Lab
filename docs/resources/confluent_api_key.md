---
# generated by https://github.com/hashicorp/terraform-plugin-docs
page_title: "confluent_api_key Resource - terraform-provider-confluent"
subcategory: ""
description: |-
  
---

# confluent_api_key Resource

<img src="https://img.shields.io/badge/Lifecycle%20Stage-Public%20Preview-%2300afba" alt="">

`confluent_api_key` provides an API Key resource that enables creating, editing, and deleting Cloud API Keys and Kafka API Keys on Confluent Cloud.

## Example Usage

### Example Kafka API Key
```terraform
resource "confluent_environment" "staging" {
  display_name = "Staging"
}

resource "confluent_kafka_cluster" "basic" {
  display_name = "inventory"
  availability = "SINGLE_ZONE"
  cloud        = "AWS"
  region       = "us-east-2"
  basic {}
  environment {
    id = confluent_environment.staging.id
  }
}

resource "confluent_service_account" "app-manager" {
  display_name = "app-manager"
  description  = "Service account to manage 'inventory' Kafka cluster"
}

resource "confluent_role_binding" "app-manager-kafka-cluster-admin" {
  principal   = "User:${confluent_service_account.app-manager.id}"
  role_name   = "CloudClusterAdmin"
  crn_pattern = confluent_kafka_cluster.basic.rbac_crn
}

resource "confluent_api_key" "app-manager-kafka-api-key" {
  display_name = "app-manager-kafka-api-key"
  description  = "Kafka API Key that is owned by 'app-manager' service account"
  owner {
    id          = confluent_service_account.app-manager.id
    api_version = confluent_service_account.app-manager.api_version
    kind        = confluent_service_account.app-manager.kind
  }

  managed_resource {
    id          = confluent_kafka_cluster.basic.id
    api_version = confluent_kafka_cluster.basic.api_version
    kind        = confluent_kafka_cluster.basic.kind

    environment {
      id = confluent_environment.staging.id
    }
  }

  # The goal is to ensure that confluent_role_binding.app-manager-kafka-cluster-admin is created before
  # confluent_api_key.app-manager-kafka-api-key is used to create instances of
  # confluent_kafka_topic, confluent_kafka_acl resources.

  # 'depends_on' meta-argument is specified in confluent_api_key.app-manager-kafka-api-key to avoid having
  # multiple copies of this definition in the configuration which would happen if we specify it in
  # confluent_kafka_topic, confluent_kafka_acl resources instead.
  depends_on = [
    confluent_role_binding.app-manager-kafka-cluster-admin
  ]
}
```

### Example Cloud API Key
```terraform
resource "confluent_environment" "staging" {
  display_name = "Staging"
}

resource "confluent_service_account" "env-manager" {
  display_name = "env-manager"
  description  = "Service account to manage resources under 'Staging' environment on Confluent Cloud"
}

resource "confluent_role_binding" "app-manager-env-admin" {
  principal   = "User:${confluent_service_account.env-manager.id}"
  role_name   = "EnvironmentAdmin"
  crn_pattern = confluent_environment.staging.resource_name
}

resource "confluent_api_key" "env-manager-cloud-api-key" {
  display_name = "env-manager-cloud-api-key"
  description  = "Cloud API Key that is owned by 'env-manager' service account"
  owner {
    id          = confluent_service_account.env-manager.id
    api_version = confluent_service_account.env-manager.api_version
    kind        = confluent_service_account.env-manager.kind
  }

  # The goal is to ensure that confluent_role_binding.app-manager-env-admin is created before
  # confluent_api_key.env-manager-cloud-api-key is used.

  depends_on = [
    confluent_role_binding.app-manager-env-admin
  ]
}
```

<!-- schema generated by tfplugindocs -->
## Argument Reference

The following arguments are supported:

- `display_name` - (Required String) A human-readable name for the API Key.
- `description` - (Optional String) A free-form description of the API Account.
- `owner` (Required Configuration Block) supports the following:
    - `id` - (Required String) The ID of the owner that the API Key belongs to, for example, `sa-abc123` or `u-abc123`.
    - `api_version` - (Required String) The API group and version of the owner that the API Key belongs to, for example, `iam/v2`.
    - `kind` - (Required String) The kind of the owner that the API Key belongs to, for example, `ServiceAccount` or `User`.
- `managed_resource` (Optional Configuration Block) This block must be set for Kafka API Keys and must be omitted for Cloud API Keys. It supports the following:
    - `id` - (Required String) The ID of the managed resource that the API Key associated with, for example, `lkc-abc123`.
    - `api_version` - (Required String) The API group and version of the managed resource that the API Key associated with, for example, `cmk/v2`.
    - `kind` - (Required String) The kind of the managed resource that the API Key associated with, for example, `Cluster`.
    - `environment` (Required Configuration Block) supports the following:
        - `id` - (Required String) The ID of the Environment that the managed resource belongs to, for example, `env-abc123`.

## Attributes Reference

In addition to the preceding arguments, the following attributes are exported:

- `id` - (Required String) The ID of the API Key, for example, `EGWX3S4BVNQIRBMJ`.
- `secret` - (Required String, Sensitive) The secret of the API Key.

-> **Note:** If human access is required, you can read out and store the `secret` attribute itself in a key vault.

## Import

-> **Note:** You must set the `API_KEY_SECRET` (`secret`) environment variable before importing an API Key.

You can import a Kafka API Key by using the Environment ID and Kafka API Key ID in the format `<Environment ID>/<Kafka API Key ID>`, for example:

```shell
$ export CONFLUENT_CLOUD_API_KEY="<cloud_api_key>"
$ export CONFLUENT_CLOUD_API_SECRET="<cloud_api_secret>"
$ export API_KEY_SECRET="<api_key_secret>"

# Option #1: Kafka API Key
$ terraform import confluent_api_key.example_kafka_api_key "env-abc123/UTT6WDRXX7FHD2GV"
```

You can import a Cloud API Key by using Cloud API Key ID, for example:

```shell
$ export CONFLUENT_CLOUD_API_KEY="<cloud_api_key>"
$ export CONFLUENT_CLOUD_API_SECRET="<cloud_api_secret>"
$ export API_KEY_SECRET="<api_key_secret>"

# Option #2: Cloud API Key
$ terraform import confluent_api_key.example_cloud_api_key "4UEXOMMWIBE5KZQG"
```

!> **Warning:** Do not forget to delete terminal command history afterwards for security purposes.

## Getting Started
The following end-to-end examples might help to get started with `confluent_api_key` resource:
  * [`basic-kafka-acls`](https://github.com/confluentinc/terraform-provider-confluent/tree/master/examples/configurations/basic-kafka-acls): _Basic_ Kafka cluster with authorization using ACLs
  * [`standard-kafka-acls`](https://github.com/confluentinc/terraform-provider-confluent/tree/master/examples/configurations/standard-kafka-acls): _Standard_ Kafka cluster with authorization using ACLs
  * [`standard-kafka-rbac`](https://github.com/confluentinc/terraform-provider-confluent/tree/master/examples/configurations/standard-kafka-rbac): _Standard_ Kafka cluster with authorization using RBAC
  * [`dedicated-public-kafka-acls`](https://github.com/confluentinc/terraform-provider-confluent/tree/master/examples/configurations/dedicated-public-kafka-acls): _Dedicated_ Kafka cluster that is accessible over the public internet with authorization using ACLs
  * [`dedicated-public-kafka-rbac`](https://github.com/confluentinc/terraform-provider-confluent/tree/master/examples/configurations/dedicated-public-kafka-rbac): _Dedicated_ Kafka cluster that is accessible over the public internet with authorization using RBAC
  * [`dedicated-privatelink-aws-kafka-acls`](https://github.com/confluentinc/terraform-provider-confluent/tree/master/examples/configurations/dedicated-privatelink-aws-kafka-acls): _Dedicated_ Kafka cluster on AWS that is accessible via PrivateLink connections with authorization using ACLs
  * [`dedicated-privatelink-aws-kafka-rbac`](https://github.com/confluentinc/terraform-provider-confluent/tree/master/examples/configurations/dedicated-privatelink-aws-kafka-rbac): _Dedicated_ Kafka cluster on AWS that is accessible via PrivateLink connections with authorization using RBAC
  * [`dedicated-privatelink-azure-kafka-rbac`](https://github.com/confluentinc/terraform-provider-confluent/tree/master/examples/configurations/dedicated-privatelink-azure-kafka-rbac): _Dedicated_ Kafka cluster on Azure that is accessible via PrivateLink connections with authorization using RBAC
  * [`dedicated-privatelink-azure-kafka-acls`](https://github.com/confluentinc/terraform-provider-confluent/tree/master/examples/configurations/dedicated-privatelink-azure-kafka-acls): _Dedicated_ Kafka cluster on Azure that is accessible via PrivateLink connections with authorization using ACLs
  * [`dedicated-vnet-peering-azure-kafka-acls`](https://github.com/confluentinc/terraform-provider-confluent/tree/master/examples/configurations/dedicated-vnet-peering-azure-kafka-acls): _Dedicated_ Kafka cluster on Azure that is accessible via VPC Peering connections with authorization using ACLs
  * [`dedicated-vnet-peering-azure-kafka-rbac`](https://github.com/confluentinc/terraform-provider-confluent/tree/master/examples/configurations/dedicated-vnet-peering-azure-kafka-rbac): _Dedicated_ Kafka cluster on Azure that is accessible via VPC Peering connections with authorization using RBAC
  * [`dedicated-vpc-peering-aws-kafka-acls`](https://github.com/confluentinc/terraform-provider-confluent/tree/master/examples/configurations/dedicated-vpc-peering-aws-kafka-acls): _Dedicated_ Kafka cluster on AWS that is accessible via VPC Peering connections with authorization using ACLs
  * [`dedicated-vpc-peering-aws-kafka-rbac`](https://github.com/confluentinc/terraform-provider-confluent/tree/master/examples/configurations/dedicated-vpc-peering-aws-kafka-rbac): _Dedicated_ Kafka cluster on AWS that is accessible via VPC Peering connections with authorization using RBAC
  * [`dedicated-vpc-peering-gcp-kafka-acls`](https://github.com/confluentinc/terraform-provider-confluent/tree/master/examples/configurations/dedicated-vpc-peering-gcp-kafka-acls): _Dedicated_ Kafka cluster on GCP that is accessible via VPC Peering connections with authorization using ACLs
  * [`dedicated-vpc-peering-gcp-kafka-rbac`](https://github.com/confluentinc/terraform-provider-confluent/tree/master/examples/configurations/dedicated-vpc-peering-gcp-kafka-rbac): _Dedicated_ Kafka cluster on GCP that is accessible via VPC Peering connections with authorization using RBAC