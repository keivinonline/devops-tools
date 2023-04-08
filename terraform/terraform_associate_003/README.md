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
## Best practices
### Data Sharing
- explicitly configuration such a `data.terraform_remote_state.vpc.outputs.subnet_id`
### Module versions
- use specific versions for modules to ensure consistency
- it also avoids unwanted updates
### Terraform core and provider versions
- use minimum versions such as `>= 0.12.0`to preven breaking incompatibilities 
- root modules should use `~>` to set lower and upper bound vesions
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