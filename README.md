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