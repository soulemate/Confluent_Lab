---
# generated by https://github.com/hashicorp/terraform-plugin-docs
page_title: "confluent_identity_pool Resource - terraform-provider-confluent"
subcategory: ""
description: |-
  
---

# confluent_identity_pool Resource

[![Limited Availability](https://img.shields.io/badge/Lifecycle%20Stage-Limited%20Availability-%2345c6e8)](https://docs.confluent.io/cloud/current/api.html#section/Versioning/API-Lifecycle-Policy) [![Request Access To OAuth API](https://img.shields.io/badge/-Request%20Access%20To%20OAuth%20API-%23bc8540)](mailto:ccloud-api-access+iam-v2-closed-preview@confluent.io?subject=Request%20to%20join%20OAuth%20API%20Closed%20Preview&body=I%E2%80%99d%20like%20to%20join%20the%20Confluent%20Cloud%20API%20Closed%20Preview%20for%20iam/v2%20to%20provide%20early%20feedback%21%20My%20Cloud%20Organization%20ID%20is%20%3Cretrieve%20from%20https%3A//confluent.cloud/settings/billing/payment%3E.)

-> **Note:** `confluent_identity_pool` resource is available in a **Limited Availability** for early adopters. Limited Availability features are introduced to gather customer feedback. This feature should be used only for evaluation and non-production testing purposes or to provide feedback to Confluent, particularly as it becomes more widely available in follow-on editions.  
**Limited Availability** features are intended for evaluation use in development and testing environments only, and not for production use. The warranty, SLA, and Support Services provisions of your agreement with Confluent do not apply to Limited Availability features. Limited Availability features are considered to be a Proof of Concept as defined in the Confluent Cloud Terms of Service. Confluent may discontinue providing preview releases of the Limited Availability features at any time in Confluent’s sole discretion.

`confluent_identity_pool` provides an Identity Pool resource that enables creating, editing, and deleting identity pools on Confluent Cloud.

## Example Usage

### Example Identity Pool to be used with Azure AD

```terraform
resource "confluent_identity_provider" "azure" {
  display_name = "My OIDC Provider: Azure AD"
  description  = "My description"
  issuer       = "https://login.microsoftonline.com/{tenant_id}/v2.0"
  jwks_uri     = "https://login.microsoftonline.com/common/discovery/v2.0/keys"
}

resource "confluent_identity_pool" "example" {
  identity_provider {
    id = confluent_identity_provider.azure.id
  }
  display_name    = "My Identity Pool"
  description     = "Prod Access to Kafka clusters to Release Engineering"
  identity_claim  = "claims.sub"
  filter          = "claims.aud==\"confluent\" && claims.group!=\"invalid_group\""
}
```

### Example Identity Pool to be used with Okta

```terraform
resource "confluent_identity_provider" "okta" {
  display_name = "My OIDC Provider: Okta"
  description  = "My description"
  issuer       = "https://mycompany.okta.com/oauth2/default"
  jwks_uri     = "https://mycompany.okta.com/oauth2/default/v1/keys"
}

resource "confluent_identity_pool" "example" {
  identity_provider {
    id = confluent_identity_provider.okta.id
  }
  display_name    = "My Identity Pool"
  description     = "Prod Access to Kafka clusters to Release Engineering"
  identity_claim  = "claims.sub"
  filter          = "claims.aud==\"confluent\" && claims.group!=\"invalid_group\""
}
```

<!-- schema generated by tfplugindocs -->
## Argument Reference

The following arguments are supported:

- `identity_provider` (Required Configuration Block) supports the following:
    - `id` - (Required String) The ID of the Identity Provider associated with the Identity Pool, for example, `op-abc123`.
- `display_name` - (Required String) A human-readable name for the Identity Pool.
- `description` - (Required String) A description for the Identity Pool.
- `identity_claim` - (Required String) The JSON Web Token (JWT) claim to extract the authenticating identity to Confluent resources from (see [Registered Claim Names](https://datatracker.ietf.org/doc/html/rfc7519#section-4.1) for more details). This appears in the audit log records, showing, for example, that "identity Z used identity pool X to access topic A".
- `filter` - (Required String) A filter expression in [Supported Common Expression Language (CEL)](https://docs.confluent.io/cloud/current/access-management/authenticate/oauth/identity-pools.html#supported-common-expression-language-cel-filters) that specifies which identities can authenticate using your identity pool (see [Set identity pool filters](https://docs.confluent.io/cloud/current/access-management/authenticate/oauth/identity-pools.html#set-identity-pool-filters) for more details).

## Attributes Reference

In addition to the preceding arguments, the following attributes are exported:

- `id` - (Required String) The ID of the Identity Pool, for example, `pool-xyz456`.

## Import

-> **Note:** `CONFLUENT_CLOUD_API_KEY` and `CONFLUENT_CLOUD_API_SECRET` environment variables must be set before importing an Identity Pool.

You can import an Identity Pool by using Identity Provider ID and Identity Pool ID, in the format `<Identity Provider ID>/<Identity Pool ID>`. The following example shows how to import an Identity Pool:

```shell
$ export CONFLUENT_CLOUD_API_KEY="<cloud_api_key>"
$ export CONFLUENT_CLOUD_API_SECRET="<cloud_api_secret>"
$ terraform import confluent_identity_pool.example op-abc123/pool-xyz456
```

!> **Warning:** Do not forget to delete terminal command history afterwards for security purposes.

## External Documentation
* [Use identity pools with your OAuth provider](https://docs.confluent.io/cloud/current/access-management/authenticate/oauth/identity-pools.html).