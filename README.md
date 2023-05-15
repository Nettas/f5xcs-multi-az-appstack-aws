# f5xcs-multi-az-appstack-aws

This is a non-official F5 repository.  This repo is not supported by F5 or DevCentral!

This repo will provide a solution for deploying F5 XCS Multi-AZ AppStack in AWS.

# Distributed Cloud AWS Multi-AZ AppStack Deployment

The goal of this solution is to provide the infrastructure for a working demo to deploy F5 Distributed Cloud AppStack in AWS in multiple availability zones.
<!--TOC-->

- [F5 Distributed Cloud AWS AppStack Multi-AZ Deployment](#f5-distribued-cloud-aws-appstack-multi-zone-deployment)
  - [To do](#to-do)
  - [High Level Topology](#topology)
  - [Requirements](#requirements)
  - [Modules](#modules)
  - [Inputs](#inputs)
  - [Deployment](#deployment)
  - [Troubleshooting](#troubleshooting)
  - [Support](#support)
  - [Community Code of Conduct](#community-code-of-conduct)
  - [License](#license)
  - [Copyright](#copyright)
  - [F5 Networks Contributor License Agreement](#f5-networks-contributor-license-agreement)

<!--TOC-->

## To do

- Optional step (if you plan to run Managed or Physical k8s):
    - Add cluster role with proper policy rules.
        - URL List
            - URLs:*
            - Allowed Verbs:*
        - Resource List
            - API Groups:*
            - Resource Types:*
            - Allowed Verbs:*
        - Resource List
            - API Groups:rbac.authorization.k8s.io
            - Resource Types:rolebindings, clusterroles, clusterrolebindings
            - Resource Instances:admin, edit, view
            - Allowed Verbs:create, bind, escalate
    - Add Cluster Role Bidning for user.  Select ves-io-admin-cluster-role.
        - Subject - email of user account
- Infrastructure buildout in AWS
    - Export variables:
        - export VOLT_API_P12_FILE=/creds/.api-creds.p12
        - export VES_P12_PASSWORD=12345678
    - Validate AWS IAM Permissions and Access and Secret Key were created
    - Create a tfvars file or override.tf
    - Manully or Auto Deploy (see [Deployment](#deployment) options below):

## Topology
- High Level Topology 

![Rough Diagram](/images/AWS-AppStack.png)

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Requirements

| Name | Version |
|------|---------|
| <a name="requirement_terraform"></a> [terraform](#requirement\_terraform) | >= 0.13 |
| <a name="requirement_google"></a> [aws](#requirement\_aws) | ~> 4.0 |
| <a name="requirement_http"></a> [http](#requirement\_http) | 2.1.1 |
| <a name="requirement_volterrarm"></a> [volterrarm](#requirement\_volterrarm) | 0.11.6 |

## Modules

| Name | Source | Version |
|------|--------|---------|
| <a name="module_util"></a> [util](#module\_util) | ./util | n/a |
| <a name="module_xcs"></a> [xcs](#module\_xcs) | ./xcs | n/a |

## Inputs

| Name | Description | Type | Default |
|------|-------------|------|---------|
| <a name="input_tenant_name"></a> [tenant\_name](#input\_tenant\_name) | REQUIRED:  This is your Volterra Tenant Name:  https://<tenant\_name>.console.ves.volterra.io/api | `string` | `"f5-xc-lab-app"` |
| <a name="input_namespace"></a> [namespace](#input\_namespace) | REQUIRED:  This is your Volterra Namespace | `string` | `"app1-dev"` |
| <a name="input_api_cert"></a> [api\_cert](#input\_api\_cert) | REQUIRED:  This is the path to the Volterra API Key.  See https://volterra.io/docs/how-to/user-mgmt/credentials | `string` | `"./creds/api2.cer"` |
| <a name="name"></a> [name](#inputs\_name) | REQUIRED:  This is your Distributed Cloud prefix name | `string` | `"cust-provided"` |
| <a name="stack_name"></a> [stackname](#inputs\_stack\_name) | REQUIRED:  This is your Distributed Cloud AppStack name | `string` | `"aws-app-stack"` |
| <a name="aws_region"></a> [aws_region](#inputs\_aws\_region) | REQUIRED:  This is your AWS Region | `string` | `"us-east-2"` |
| <a name="aws_region_one_zone_a"></a> [aws_region_one_zone_a](#inputs\_aws\_region\_one\_zone\_a) | REQUIRED:  This is your AWS Region One AZ A| `string` | `"us-east2a"` |
| <a name="aws_region_one_zone_b"></a> [aws_region_one_zone_b](#inputs\_aws\_region\_one\_zone\_b) | REQUIRED:  This is your AWS Region One AZ B| `string` | `"us-east2b"` |
| <a name="aws_region_one_zone_c"></a> [aws_region_one_zone_c](#inputs\_aws\_region\_one\_zone\_c) | REQUIRED:  This is your AWS Region One AZ C| `string` | `"us-east2c"` |
| <a name="aws_instance_type"></a> [aws_instance_type](#inputs\_aws\_instance\_type) | REQUIRED:  This is your AWS Instance Type | `string` | `"n1-stnadard-4"` |
| <a name="input_sshPublicKey"></a> [sshPublicKey](#input\_sshPublicKey) | OPTIONAL: ssh public key for instances | `string` | `""` |
| <a name="input_api_p12_file"></a> [api\_p12\_file](#input\_api\_p12\_file) | REQUIRED:  This is the path to the Volterra API Key.  See https://volterra.io/docs/how-to/user-mgmt/credentials | `string` | `"./creds/f5-xc-lab-app.console.ves.volterra.io.api-creds.p12"` |
| <a name="input_sshPublicKeyPath"></a> [sshPublicKeyPath](#input\_sshPublicKeyPath) | OPTIONAL: ssh public key path for instances | `string` | `"./creds/id_rsa.pub"` |
| <a name="input_api_key"></a> [api\_key](#input\_api\_key) | REQUIRED:  This is the path to the Volterra API Key.  See https://volterra.io/docs/how-to/user-mgmt/credentials | `string` | `"./creds/api.key"` |
| <a name="input_delegated_dns_domain"></a> [delegated\_dns\_domain](#input\_delegated\_dns\_domain) | n/a | `string` | `"user-defined"` |
| <a name="input_volterra_tf_action"></a> [volterra\_tf\_action](#input\_volterra\_tf\_action) | n/a | `string` | `"apply"` |
| <a name="input_gateway_type"></a> [gateway\_type](#input\_gateway\_type) | n/a | `string` | `"voltstack_cluster"` |
| <a name="api_url"></a> [api\_url](#input\_api\_url) | n/a | `string` | `"https://tenant-name.console.ves.volterra.io/api"` |
<a name="input_tags"></a> [tags](#input\_tags) | Environment tags for objects | `map(string)` | <pre>{<br>  "application": "f5app",<br>  "costcenter": "f5costcenter",<br>  "creator": "Terraform",<br>  "delete": "True",<br>  "environment": "gcp",<br>  "group": "f5group",<br>  "owner": "f5owner",<br>  "purpose": "public"<br>}</pre> | 

## Deployment

For manual deployment you can do the traditional terraform commands.

```bash
terraform init
terraform plan
terraform apply --auto-approve
```

For auto deployment you can do with the deploy.sh and destroy.sh scripts.

```bash
./deploy
./destroy
```

## Troubleshooting

Please refer to the following: 
- F5 Distributed Cloud
    - [F5 Distributed Cloud](https://docs.cloud.f5.com/docs/)
- Terraform
    - [F5 Distributed Cloud Terraform Registry](https://registry.terraform.io/providers/volterraedge/volterra/latest/docs).

## Support

For support, please open a GitHub issue.  Note, the code in this repository is community supported and is not supported by F5 Networks.  For a complete list of supported projects please reference [SUPPORT.md](SUPPORT.md).

## Community Code of Conduct

Please refer to the [F5 DevCentral Community Code of Conduct](code_of_conduct.md).

## License

[Apache License 2.0](LICENSE)

## Copyright

Copyright 2014-2022 F5 Networks Inc.

### F5 Networks Contributor License Agreement

Before you start contributing to any project sponsored by F5 Networks, Inc. (F5) on GitHub, you will need to sign a Contributor License Agreement (CLA).

If you are signing as an individual, we recommend that you talk to your employer (if applicable) before signing the CLA since some employment agreements may have restrictions on your contributions to other projects.
Otherwise by submitting a CLA you represent that you are legally entitled to grant the licenses recited therein.

If your employer has rights to intellectual property that you create, such as your contributions, you represent that you have received permission to make contributions on behalf of that employer, that your employer has waived such rights for your contributions, or that your employer has executed a separate CLA with F5.

If you are signing on behalf of a company, you represent that you are legally entitled to grant the license recited therein.
You represent further that each employee of the entity that submits contributions is authorized to submit such contributions on behalf of the entity pursuant to the CLA.