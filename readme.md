This will build, package, and publish the helm chart to artifactory.


# INPUTS

`target_ref`: (Optional) The target branch ref that the version number change is pushed to. (default `develop`)  
`lint_args`: (Optional) Additional arguments to pass to `helm lint` (default `''`)  
`GITHUB_TOKEN`: (Optional) github token (default `github.token`)  
`ARTIFACTORY_URL`: (Required) URL for artifactory.  
`ARTIFACTORY_USER`: (Required) Artifactory login user  
`ARTIFACTORY_TOKEN`: (Required) Artifactory login api token  
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
      - uses: camalot/action-helm-build-publish@v1
        id: helm_build
        with:
          ARTIFACTORY_URL: "${{ secrets.SRE_ARTIFACTORY_URL }}"
          ARTIFACTORY_USER: "${{ secrets.SRE_ARTIFACTORY_USER }}"
          ARTIFACTORY_TOKEN: "${{ secrets.SRE_ARTIFACTORY_TOKEN }}"
          GITHUB_TOKEN: "${{ github.token }}"
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
