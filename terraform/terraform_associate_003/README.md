# terraform_associate_003
## Review terraform fundamentals
### Terraform state
1. purpose
- state is a hard requirement
- in cases where state is not required, it requires complex shift from state to replacement concept
2. mapping to real world
- uses own state structure to map to real world
- expects each remote object ot bound to only 1 resource instance
- unexpected actions may be taken if mapping is 1 to many 
3. Metadata
- maps between resources and remote objects
- also tracks resource dependencies
- tf retains copy of most recent set of dependencies within th state which is used to determine the destruction order when item(s) are deleted from the configuration
- tf also stores pointers to the provider configuration that was recently used where multiple providers are present
4. Performance
- tf stores cache of attribute values for all resources for better performance
- large users can run `-refresh=false` and `-target` flags to work around rate limiting issues
5. syncing
- by default, tf stores state in a file in current working directory as `terraform.tfstate`
- `Remote state` is recommended for teams
    - `Remote locking` can also be used to prevent brain split
    - `terraform_remote_state` also allows output values to be shared 
    - can be used with consul to share general stores of data
### Terraform settings
1. terraform block syntax
```hcl
terraform {
    ...
}
```
- only constant values can be usd in the block
- `cloud` block is for Terraform Cloud
2. Backend
- via `backend` block
3. specifying required terraform version
- accepts a version constraint string
~> 1.0.4 means up to 1.0.5 or 1.0.10 etc but NOT 1.1.0 AKA pessimistic contraint operator
- if no constraint is specified, tf will use latest version and if that fails, operations will fail e.g. plan will fail
- prerelease versions such as `1.2.0-beta` can only be used when it is exact match `=` and not `~>` or `>=`
- `required_version` only applies to the CLI.
- `required_providers` only applies to the providers.
```hcl
terraform {
    required_providers {
        aws = {
            version = ">= 2.7.0"
            source = "hashicorp/aws"
        }
    }
}
```
4. experimental features
- opt-in to experimental features
```hcl
terraform {
    experiments = [example]
}
```
- any module that uses experimental features will produce a WARNING on every plan and apply
- recommended to explore in `alpha` and `beta` releases of the modules
5. passing metadata to providers
- `provider_meta` block is a nested block within a module
- mainly used for modules distributed by same vendor or provider
### Manage terraform versions
- in general, tf will work across minor versions updates (i.e. major.minor.patch semantic)
- tf will update state file version if required and warn if not compatible
- updating state file version is 1 way (i.e. cannot downgrade)
### Providers
- interact with remote APIs
- each provider adds a set of resources types and/or data sources for TF to manage
- TF CLI finds and installs provides when init is done on a directory
- use `plugin_cache_dir` to save time and bandwidth when running `terraform init` locally or CICD
- can use with `dependency_lock_file` and commit to Git to ensure consistency
### How terraform works with plugins
- terraform consists of 
    - core
    - plugins
- TF uses RPC to communicate with plugins
1. TF core
- statically compiled binary in Go
2. TF plugins
- written in Go and invoked by TF core over RPC
3. Responsibilities of Provider Plugins
- initialize libraries to make API calls
- authenticate with infa provider
- define resources that map to specific services
4. Selecting plugins
- after running terraform init
- saves into `.terraform/providers/` directory
5. upgrading plugins
- use `terraform init -upgrade` to upgrade plugins
- only applicable to the `.terraform/providers/` directory
### Provider Configuration
- must be in root module 
```hcl
// becomes the default configuration for the provider
provider "aws"{
    region = "ap-southeast-1"
}
provider "aws" {
    alias = "west"
    region = "us-west-2"
}
// to use
terraform {
    required_providers {
        aws = {
            source = "hashicorp/aws"
            configuration_aliases = [aws.west]
        }
    }
}
// selecting alternate provider config
resource " aws_instances" "foo" {
    provider = aws.west
}
// for child modules
module "aws_vpc" {
    source = "./aws_vpc"
    providers = {
        aws = aws.west
    }
}
```
#### Dependency Lock File
- introduced in version 0.14
- `terraform.lock.hcl`
    - contains a list of all providers and their versions
- should check this file into GIT
- only track provider versions and NOT module versions 
1. lock file location
- belongs to the configuration as a whole 
- `terraform.lock.hcl` is updated only when performing init
2. dependency installation behavior
- check for version recorded in lock file first and reuse
  - can override with `terraform init -upgrade` flag
- else will get latest version that matches the constraint
- `terraform init` will warn if any changes are made to the lock file
- `h1` and `zh` are different hashing schemas 
- `zh` is for zip hash 
- `h1` is for hash scheme 1 
- TF will opportunistically add `h1` checksums 
- can use the command below to avoid ongoing addition of new `h1` hashses 
```bash
terraform providers lock \ 
    -platform=linux_amd64 \ 
    -platform=darwin_amd64 \ 
    -platform=windows_amd64
```
- this will download and verify all official packages

## Navigate core workflow
### Core workflow
1. write
- author of infra as code
2. plan 
- preview before applying
3. apply
- provision resources
## Commands - Init
- can run `-from-module=MODULE_SOURCE` to copy the module source into the current directory before init
    - can be used in a VCS flow
- `-migrate-state` copies existing state to new backend
  - `-force-copy` suppresses command prompt to confirm
- `-reconfigure` disregards any existing backend configuration and prevents migration of any existing state
- `-backend=false` to skip backend initialization
- `-backend-config=...` for partial backend configuration
## Commands - Plan
- reads current state of any existing remote objects to check if it's up to date
- compare current config to previous state
- proposes set of change actions
- can use `-out=FILE` opton to save generated plan 
- `out` file format is in binary but can be saved as JSON with `terraform show -json FILE`
### Planning modes
- activating any of the following disabled `normal` mode
1. destroy mode (for both plan and apply for v0.15 and above)
- `-destroy` flag
2. Refresh-only mode (v0.15 and above)
- to reconcile TF records with changes from remote
- `-refresh-only` flag
### Planning options
1. `-refresh=false` 
- TF ignores external changes
2. `-replace=ADDRESS`
3. `-target=ADDRESS`    
4. `-var 'NAME=VALUE` 
5. `-var-file=FILENAME` from a tfvars file
### Other options
1. `-compat-warnings`
- only shows summary of warnings
2. `-detailed-exitcode`
- 0 if no changes and, 1 for error and 2 for non-empty diff(changes present)
### Plan details
1. `.prioir_state`
- contains state prior to plan action
2. Review resource drift
- `.resource_drift` indicates changes outside of TF workflow
## Commands - Apply
### Automatic plan mode
- by default it runs plan before apply
### Plan options
- when running without a saved plan, apply supports all planning modes
## Apply TF configuration
- if an existing lock file `.terraform.tfstate.lock.info` is present, it will report an error and exit
### Errors during apply
1. logs error and reports to console
2. updates state file with changes
3. unlocks state file 
4. exits
- TF does not support rolling back a partially-completed apply
## Commands - destroy
- remove all remote objects managed by TF
- `terraform destroy` is an alias for `terraform apply -destroy` (since v0.15)
- hence it accepts most of the options that apply to `terraform apply`
## Commands - fmt
- `-list=false` don't list files containing inconsistencies
- `-write=false` dont overwrite input files
- `-diff` to show diffs 
- `-check` to check if formatting is required
- `-recursive`, by default it only formats the current directory
## Best practices
### Data Sharing
- explicitly configuration such a `data.terraform_remote_state.vpc.outputs.subnet_id`
### Module versions
- use specific versions for modules to ensure consistency
- it also avoids unwanted updates
### Terraform core and provider versions
- use minimum versions such as `>= 0.12.0`to preven breaking incompatibilities 
- root modules should use `~>` to set lower and upper bound vesions
### Plans
- never commit a plan file to a VCS as it contains sensitive information
- applying saved plans do not prompt for approvals
### .tfvars
- tfvars should not be committed to VCS especially if they contain sensitive information
## enable logging 
- `export TF_LOG_CORE=TRACE`
- `export TF_LOG_PROVIDER=TRACE`
- `export TF_LOG_PATH=logs.txt`

## Terraform Cloud
### Remote state 
- terraform cloud uses stronger locking concepts that detect attempts to create a new plan when existing plan is waiting approval, by queueing the operations
- `tfe_outputs` is more secure as it does not required full access to workspace state
### Cloud configuration
- enables CLI-driven run workflow
### Providers
- installs providers on every run 
### Core workflow enhanced by TF cloud
1. write
- centralized and secure location for input vars and state
- single TF cloud API key for team members 
2. plan
- auto run plans on every commit
### Sensitive data
- always encrypts state at rest and protects in transit with TLS
- also supports detailed audit logging 
### refresh-only mode
- `allow-empty-apply` can be set to True in TF cloud
- Actions > start new plan > allow empty apply run type

## Things to check on 
- [ ] tf cloud remote state
- [ ] tf workspace
- [ ] tf cloud capabilities
- [ ] try tf cloud ? 
- [ ] provider_meta
- [ ] reread https://developer.hashicorp.com/terraform/language/v1.1.x/files/dependency-lock
- [ ] https://developer.hashicorp.com/terraform/tutorials/automation/automate-terraform?utm_source=WEBSITE&utm_medium=WEB_IO&utm_offer=ARTICLE_PAGE&utm_content=DOCS
- [ ] https://developer.hashicorp.com/terraform/language/v1.1.x/settings/backends/configuration#partial-configuration

https://developer.hashicorp.com/terraform/cli/cloud



## additional commands
- `terraform get` to get new versions of modules
- `terraform validate` to validate the configuration files in a directory

## folders
- `.terraform` contains
    - `plugins` contains the provider plugins
    - `modules` contains the modules
    - `terraform.tfstate` contains the state file
- stuff in `.terraform` is read-only and should not be modified
- `modules.json`
```json
{
  "Modules": [
    {
      "Key": "nw_test_public",
      "Source": "registry.terraform.io/terraform-aws-modules/ec2-instance/aws",
      "Version": "3.6.0",
      "Dir": ".terraform/modules/nw_test_public"
    },
```

## Objective - #7
### Manage State
1. State locking
- happens during write operations
- can be disabled with `-lock=false` flag
- can use `terraform force-unlock LOCK_ID` to unlock a state file
- lock ID act as a nonce to for unique target
2. Protect sensitive input vars
- use `sensitive=true` to mark a var as sensitive and this will prevent it from being logged in console
- can use a separate `secret.tfvars` or better use `TF_VAR_<var_name>` to decalre during cli run 
- to get output from sensitive values, use the 
```hcl
output "db_connect_string" {
    value = ... 
    sensitive = true
}
```
3. sensitive data in state
- as of v0.9, TF does not persist data locally when using remote state
### Backend Management
1. Command - login
- obtain and save API token for TF cloud or TF enterprise
- `terraform login <hostname>`
  - uses `app.terraform.io` by default
  - saves in `.terraform.d/credentials.tfrc.json`
2. Backends
- stores state file
- defaults to local 
3. local backends
```hcl
terraform {
    backend "local" {
        path = "path/to/terraform.tfstate"
    }
}
data "terraform_remote_state" "foo" {
    backend = "local"
    config = {
        path = "path/to/terraform.tfstate"
    }
}
```
4. backend configuration
- including access credentials in the backend configuration is not recommended
- providing creds via env files is better
- for consul
```hcl
terraform {
    backend "consul" {}
}
```
- the backend config will be
```hcl
address = "demo.consul.io"
path = "example/terraform_state"
scheme = "https"
```
- consul access token is also needed for this
```bash
CONSUL_HTTP_TOKEN=xxx
# or 
CONSUL_HTTP_AUTH=xxx
```
## Objective - #8
### Resources
- has meta-arguments
1. `depends-on`
- for hidden dependencies
```hcl
  depends_on = [
    aws_iam_role_policy.example,
  ]
```
2. `count`
- for multiple reousrces
3. `for_each`
- based on map or set of strings
```hcl
resource "azurerm_resource_group" "rg" {
    for_each = {
        a_group = "eastus"
        another_group = "westus2"
    }
    name = each.key
    location = each.value
}
```
- chaining 
```hcl
resource "aws_vpc" "foo" {
    for_each = var.vpcs
    cidr_block = each.value.cidr_block
}
resource "aws_internet_gateway" "example" {
    for_each = aws_vpc.foo
    vpc_id = each.value.id
}
```
4. `provider`
5. `lifecycle`
```hcl
...
    lifecycle {
        create_before_destroy = true
        ignore_changes = [
            tags,
        ]
        prevent_destroy = true
    }
```
6. `provisioner`
- operation time-outs
```hcl
...
    timeouts {
        create = "60m"
        delete = "2h"
    }
```
### Complex types
1. collection
- multiple values one other type
    - list(string)
    - list(number)
    - list(bool)
2. structural types
- object(...)
- tuple(...)
```hcl
{
    name = "john"
    age = 52
}
```
3. Dynamic types - "any" constraint
- `any`is not a type, but a placeholder
- list(any)
```hcl
variable "no_type_constraint" {
    type = any
}
```
4. optional object type attribute
```hcl
variable "with_optional_att" {
    type = object({
        a = string
        b = optional(string)
        c = optional(number, 123) # default value
    })
}
```
### Inject secrets into TF using vault provider
- export the env values
```bash
export VAULT_ADDR=https://vault.example.com
export VAULT_TOKEN=xxx
# successful vault msg
[INFO]  core: successful mount: namespace= path=dynamic-aws-creds-vault-admin-path/ type=aws

```
```hcl
data "vault_aws_access_credentials" "creds" {
  backend = data.terraform_remote_state.admin.outputs.backend
  role    = data.terraform_remote_state.admin.outputs.role
}
provider "aws" {
  region     = var.region
  access_key = data.vault_aws_access_credentials.creds.access_key
  secret_key = data.vault_aws_access_credentials.creds.secret_key
}
```
- every TF run will use unique set of IAM credentials
- after 120seconds, vault will destroy the short-lived AWS credentials
- benefits
    1. vault's admin responsible for IAM creds is reduced
    2. management of role permissions through vault role

## Sentinel
- "*.sentinel" file 
- types
1. hard-mandatory - halts run if policy fails
2. soft-manadatory - allows Admin to override policy failures 
3. advisory
- never interupts the run and only surface policy failures
```hcl
policy "allowed-terraform-version" {
    enforecement-level = "soft_mandatory"
}
```