# FireClover Secrets Scanner

## General Usage

```
on:
  push:
    branches:
      - main
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
      with:
        fetch-depth: 0
    - name: Secret Scanning
      uses: fc-actions/scan-secrets@v0.0.1
      with:
        extra_args: --results=verified,unknown
```

In the example config above, we're scanning for live secrets in all PRs and Pushes to `main`. Only code changes in the referenced commits are scanned. If you'd like to scan an entire branch, please see the "Advanced Usage" section below.

### Shallow Cloning

If you're incorporating TruffleHog into a standalone workflow and aren't running any other CI/CD tooling alongside TruffleHog, then we recommend using [Shallow Cloning](https://git-scm.com/docs/git-clone#Documentation/git-clone.txt---depthltdepthgt) to speed up your workflow. Here's an example for how to do it:

```
...
      - shell: bash
        run: |
          if [ "${{ github.event_name }}" == "push" ]; then
            echo "depth=$(($(jq length <<< '${{ toJson(github.event.commits) }}') + 2))" >> $GITHUB_ENV
            echo "branch=${{ github.ref_name }}" >> $GITHUB_ENV
          fi
          if [ "${{ github.event_name }}" == "pull_request" ]; then
            echo "depth=$((${{ github.event.pull_request.commits }}+2))" >> $GITHUB_ENV
            echo "branch=${{ github.event.pull_request.head.ref }}" >> $GITHUB_ENV
          fi
      - uses: actions/checkout@v3
        with:
          ref: ${{env.branch}}
          fetch-depth: ${{env.depth}}
      - uses: fc-actions/scan-secrets@v0.0.1
        with:
          extra_args: --results=verified,unknown
...
```

Depending on the event type (push or PR), we calculate the number of commits present. Then we add 2, so that we can reference a base commit before our code changes. We pass that integer value to the `fetch-depth` flag in the checkout action in addition to the relevant branch. Now our checkout process should be much shorter.

### Canary detection

TruffleHog statically detects [https://canarytokens.org/](https://canarytokens.org/) and lets you know when they're present without setting them off. You can learn more here: [https://trufflesecurity.com/canaries](https://trufflesecurity.com/canaries)

![image](https://github.com/trufflesecurity/trufflehog/assets/52866392/74ace530-08c5-4eaf-a169-84a73e328f6f)

### Advanced Usage

```yaml
- name: TruffleHog
  uses: fc-actions/scan-secrets@v0.0.1
  with:
    # Repository path
    path:
    # Start scanning from here (usually main branch).
    base:
    # Scan commits until here (usually dev branch).
    head: # optional
    # Extra args to be passed to the trufflehog cli.
    extra_args: --log-level=2 --results=verified,unknown
```

If you'd like to specify specific `base` and `head` refs, you can use the `base` argument (`--since-commit` flag in TruffleHog CLI) and the `head` argument (`--branch` flag in the TruffleHog CLI). We only recommend using these arguments for very specific use cases, where the default behavior does not work.

#### Advanced Usage: Scan entire branch

```
- name: scan-push
        uses: fc-actions/scan-secrets@v0.0.1
        with:
          base: ""
          head: ${{ github.ref_name }}
          extra_args: --results=verified,unknown
```
