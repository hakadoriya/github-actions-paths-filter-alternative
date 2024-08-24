# GitHub Actions `paths` Filter Alternative

A workaround workflow to resolve the incompatibility of the `Require status checks to pass` setting when `paths` filter is set for push trigger or pull_request trigger in GitHub Actions.

## `paths` example

```yml
name: example

on:
  push:
    # NO paths filter
  pull_request:
    # NO paths filter
  workflow_dispatch:

jobs:
  paths-filter:
    runs-on: ubuntu-latest
    outputs:
      skip: ${{ steps.paths-filter.outputs.skip }}
    steps:
      - uses: hakadoriya/github-actions-paths-filter-alternative@main
        id: paths-filter
        with:
          paths: |-
            # If any of these regular expressions match, it returns skip=false
            # This setting takes precedence over paths-ignore
            ^action.yml
  # > Note: A job that is skipped will report its status as "Success".
  # > It will not prevent a pull request from merging, even if it is a required check.
  # ref. https://docs.github.com/en/actions/using-jobs/using-conditions-to-control-job-execution#overview
  not-skip:
    runs-on: ubuntu-latest
    needs: paths-filter
    if: ${{ needs.paths-filter.outputs.skip != 'true' }}
    steps:
      - name: "if not skip"
        run: |
            echo "needs.paths-filter.outputs.skip: ${{ needs.paths-filter.outputs.skip }}"
  skip:
    runs-on: ubuntu-latest
    needs: paths-filter
    if: ${{ needs.paths-filter.outputs.skip == 'true' }}
    steps:
      - name: "if skip"
        run: |
            echo "needs.paths-filter.outputs.skip: ${{ needs.paths-filter.outputs.skip }}"
```

## `paths-ignore` example

```yml
name: example

on:
  push:
    # NO paths-ignore
  pull_request:
    # NO paths-ignore
  workflow_dispatch:

jobs:
  paths-filter:
    runs-on: ubuntu-latest
    outputs:
      skip: ${{ steps.paths-filter.outputs.skip }}
    steps:
      - uses: hakadoriya/github-actions-paths-filter-alternative@main
        id: paths-filter
        with:
          paths-ignore: |-
            # substrings of file paths to ignore written in regular expressions
            ^.github/
            ^.*\.md$
            ^.*\.log$
  # > Note: A job that is skipped will report its status as "Success".
  # > It will not prevent a pull request from merging, even if it is a required check.
  # ref. https://docs.github.com/en/actions/using-jobs/using-conditions-to-control-job-execution#overview
  not-skip:
    runs-on: ubuntu-latest
    needs: paths-filter
    if: ${{ needs.paths-filter.outputs.skip != 'true' }}
    steps:
      - name: "if not skip"
        run: |
            echo "needs.paths-filter.outputs.skip: ${{ needs.paths-filter.outputs.skip }}"
  skip:
    runs-on: ubuntu-latest
    needs: paths-filter
    if: ${{ needs.paths-filter.outputs.skip == 'true' }}
    steps:
      - name: "if skip"
        run: |
            echo "needs.paths-filter.outputs.skip: ${{ needs.paths-filter.outputs.skip }}"
```
