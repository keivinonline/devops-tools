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
## Things to check on 
- [ ] tf cloud remote state
- [ ] tf workspace
- [ ] tf cloud capabilities
- [ ] try tf cloud ? 
- [ ] provider_meta

https://developer.hashicorp.com/terraform/cli/cloud



