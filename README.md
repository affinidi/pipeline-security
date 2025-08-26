# Pipeline-security
This repository contains the various security actions required within the workflows of the Affinidi projects .

## Scanners 
The following scanners have been enabled in this repo and should be called from the desired repo to trigger the security scanning tools.

* Secret Detection : Gitleaks is used to find any potential secrets,passwords,keys etc that may have been committed by accident. 

## Usage
In your repo, ensure that you add the following code snippet to call the security scanners:

````
 jobs:
  call-security-scanners-workflow:
    uses: affinidi/pipeline-security/.github/workflows/security-scanners.yml@main
    with:
        config-path: .github/labeler.yml
    secrets: inherit    
````

For enabling wizcli-scanner, add the following code snippet to .github/workflows/on-push.yaml in your repo:

````
  # Run pipeline in context of branch, but with action config from main for opened and rebased mr's
  # also run on  branch main

  name: Wiz Scanner

  on:
  push:
    branches:
      - main
  pull_request_target:
    types:
      - opened
      - synchronize

  jobs:
    call-workflow:
      uses: affinidi/pipeline-security/.github/workflows/wizcli-dirscan.yml@main
      secrets: inherit
````

For enabling dart-scanner, add the following code snippet to .github/workflows/on-push.yaml in your repo:

````
  # Run pipeline in context of branch, but with action config from main for opened and rebased mr's
  # also run on  branch main

  name: Dart Scanner

  on:
    push:
      branches:
        - main
    pull_request:
      types:
        - opened
        - synchronize

  jobs:
    dart-security-scan:
      uses: affinidi/pipeline-security/.github/workflows/dart-scanner.yml@main
      secrets: inherit
````

Ensure the following secrets are available in your repo:

  - SNYK_SCANNER_TOKEN
  - SNYK_SCANNER_REGION
  - SNYK_GLOBAL_POLICY

For enabling rust-scanner, add the following code snippet to .github/workflows/on-push.yaml in your repo:

````
  # Run pipeline in context of branch, but with action config from main for opened and rebased mr's
  # also run on  branch main
  
  name: Rust Scanner

  on:
    push:
      branches:
        - main
    pull_request:
      types:
        - opened
        - synchronize

  jobs:
    rust-security-scan:
      uses: affinidi/pipeline-security/.github/workflows/rust-scanner.yml@main
      secrets: inherit
````

Ensure the following secrets are available in your GitHub org or at repo:

  - SNYK_SCANNER_TOKEN
  - SNYK_SCANNER_REGION
  - SNYK_GLOBAL_POLICY

---

## Unblocking Vulnerabilities

The scanners use [Snyk](https://snyk.io) under the hood to detect vulnerabilities.  

Sometimes, you may need to **ignore specific CVEs** in order to unblock a pipeline while remediation is planned.  

There are two supported ways to do this:

### 1. Local Ignore ( per repository )

If the CVE is **specific to your repository**, create or update a `.snyk` file at the root of your repo:

```bash
# Authenticate with Snyk CLI first
snyk auth $YOUR_SYNK_TOKEN

# Add an ignore for the CVE
snyk ignore --id=CVE-2024-12345 --expiry=2025-12-31 --reason="Planned upgrade in Q4"
```

This will add an entry like the following to your `.snyk` file:

```yaml
ignore:
  "CVE-2024-12345":
    - '*':
        reason: Planned upgrade in Q4
        expires: 2025-12-31T00:00:00.000Z
        created: 2025-08-26T07:35:47.825Z
```

> ℹ️ Local ignores apply **only to your repository** and are version-controlled.

---

### 2. Global Ignore ( shared across repos )

If the CVE affects **many projects**, update the global `.snyk-global` policy file in this repository [`pipeline-security`](https://github.com/affinidi/pipeline-security).

Steps:
1. Edit the [`.snyk-global`](./.snyk-global) file in this repo.
2. Add the CVE under the `ignore` section, with a reason and expiry date.
3. Commit and open a Pull Request for review.

Example entry in `.snyk-global`:

```yaml
ignore:
  "CVE-2024-56789":
    - '*':
        reason: Affects multiple Rust crates, fix pending from upstream
        expires: 2026-01-15T23:59:59.000Z
        created: 2025-08-25T10:30:00.000Z
```

Once merged, the global policy will be automatically fetched by both dart and rust scanners in future runs.

---

### Notes

- Always prefer **local ignores** if the CVE is repo-specific.
- Use **global ignores** sparingly, only for issues impacting multiple repositories.
- Every ignore must include:
  - A **reason** (why it is safe to ignore temporarily).
  - An **expiry date** (after which the CVE will be reported again).
