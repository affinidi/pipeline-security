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
    uses: affinidi/pipeline-security/.github/workflows/security-scanners.yml
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
  ```
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
      uses: affinidi/pipeline-security/.github/actions/dart-scanner@main
      secrets: inherit
  ```

  Ensure the following secrets are available in your repo:
  
  - DART_SCANNER_TOKEN
  - DART_SCANNER_REGION
