# INPUTS

`target_ref`: The target branch ref that the version number change is pushed to.
`lint_args`: Additional arguments to pass to `helm lint`

# OUTPUTS

`chart_version`: The chart version
`chart_tag`: The version tag
`chart_name`: The name of the updated chart

# REQUIRED REPO/ORG SECRETS

This secrets **MUST** exist at either the repo level or the organization level

`SRE_ARTIFACTORY_USER`: The artifactory user
`SRE_ARTIFACTORY_TOKEN`: The artifactory api token