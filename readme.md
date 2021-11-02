This will build, package, and publish the helm chart to artifactory.

- Gets the latest version from the repository tags
- Sets the next version
- Updates the `Chart.yaml`
- Commits the change to the `target_ref`
- Package the helm chart
- Pushes the package to artifactory


# INPUTS

`target_ref`: (Optional) The target branch ref that the version number change is pushed to. (default `develop`)  
`lint_args`: (Optional) Additional arguments to pass to `helm lint` (default `''`)
`lint_strict`: (Optional) Should it use `--strict` flag for linting (default `true`)
`template_lint`: (Optional) Should it use `helm template lint` vs `helm lint` (default `false`)
`plugin_version`: (Optional) The version of the helm push artifactory plugin (default `1.0.1`)
`GITHUB_TOKEN`: (Optional) github token (default `github.token`)  
`ARTIFACTORY_URL`: (Required) URL for artifactory.  
`ARTIFACTORY_USER`: (Required) Artifactory login user  
`ARTIFACTORY_TOKEN`: (Required) Artifactory login api token  
`ARTIFACTORY_REPO_NAME`: (Optional) The name of the helm repository in artifactory (default `helm`)

# OUTPUTS

`chart_version`: The chart version
`chart_tag`: The version tag
`chart_name`: The name of the updated chart

# EXAMPLES

This will update the chart version, push the changed version to the repo, package the chart, and push it to artifactory.

```yaml
name: Build Helm Chart
on:
  pull_request:
    branches:
      - develop
    types:
      - closed
jobs:
  build:
    if: github.event.pull_request.merged == true
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
        with:
          ref: develop

      # Perform the build
      - uses: camalot/action-helm-build-publish@v1.0.2
        id: helm_build
        with:
          ARTIFACTORY_URL: "${{ secrets.SRE_ARTIFACTORY_URL }}"
          ARTIFACTORY_USER: "${{ secrets.SRE_ARTIFACTORY_USER }}"
          ARTIFACTORY_TOKEN: "${{ secrets.SRE_ARTIFACTORY_TOKEN }}"
          GITHUB_TOKEN: "${{ github.token }}"

      # Comment on the PR that the chart was published 
      - uses: actions/github-script@0.9.0
        if: github.event_name == 'pull_request'
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          script: |
            const output = `:robot: I have pushed chart \`${{ steps.helm_build.outputs.chart_name }}\` version \`${{ steps.helm_build.outputs.chart_version }}\` to artifcatory.`;

            github.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: output
            })
```
