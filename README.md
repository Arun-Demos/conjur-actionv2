# CyberArk Conjur Secret Fetcher

GitHub Action for secure secrets delivery to your workflow test environment using CyberArk Conjur. Does not require a docker image to run.

Supports authenticating with CyberArk Conjur using host identity and JWT authentication.

## Requirements

* Github
* Github self hosted Runner
* Conjur Secrets Manager Enterprise v10+
* Conjur Secrets Manager Open Source v1.1+

### JWT Authentication configuration for .github/workflows .yml file

### Example

```yaml
on: [push]

permissions:
  id-token: write
  contents: read

env:
  CONJUR_URL: https://<tenant-name>.secretsmgr.cyberark.cloud/api
  CONJUR_ACCOUNT: conjur
  CONJUR_AUTHN_ID: conjur-authn-service-ID
  SECRET_ID: secret/variable/in/conjur

jobs:
  get-secrets:
    runs-on: <runner-name>
    steps:
      - uses: actions/checkout@v4

      - name: Import secrets from Conjur
        uses: Arun-Demos/conjur-actionv2@main
        with:
          url:      ${{ env.CONJUR_URL }}
          account:  ${{ env.CONJUR_ACCOUNT }}
          authn_id: ${{ env.CONJUR_AUTHN_ID }}
          # Map Conjur IDs -> env vars used later in the job
          secrets: |
            ${{ env.SECRET_ID }}|PIPELINE_ENV_VAR
      # ...
```

### Arguments

#### Required

* `url` - this is the path to your Conjur instance endpoint.  e.g. `https://conjur.cyberark.com:8443`
* `account` - this is the account configured for the Conjur instance during deployment.
* `authn_id` - this is the ID of Authn-JWT at Conjur
* `secrets` - a semi-colon delimited list of secrets to fetch.  Refer to [Secrets Syntax](#secrets-syntax) in the README below for more details.

#### Optional

* `certificate` - if using a self-signed certificate, provide the contents for validated SSL.

#### Not required
* `host_id` - this is the Host ID granted to your application by Conjur when created via policy. e.g. `host/db/github_action`
* `api_key` - this is the API key associated with your Host ID declared previously.
* `authn_token_file` - this is the file path for the Conjur auth token.

## Host Identity
### API Key Based Authentication configuration for .github/workflows .yml file
### Make sure to create secrets for Conjur Host ID and API Key in GitHub

### Example

```yaml
on: [push]
on: [push]

permissions:
  id-token: write
  contents: read

env:
  CONJUR_URL: https://<tenant-name>.secretsmgr.cyberark.cloud/api
  CONJUR_ACCOUNT: conjur
  SECRET_ID: secret/variable/in/conjur

jobs:
  test:
    # ...
    steps:
      # ...
      - name: Import Secrets using CyberArk Conjur Secret Fetcher Action
        uses: cyberark/conjur-action@v2.0.5
        with:
          url:      ${{ env.CONJUR_URL }}
          account:  ${{ env.CONJUR_ACCOUNT }}
          host_id: ${{ secrets.CONJUR_USERNAME }}
          api_key: ${{ secrets.CONJUR_API_KEY }}
          # Map Conjur IDs -> env vars used later in the job
          secrets: |
            ${{ env.SECRET_ID }}|PIPELINE_ENV_VAR
      # ...
```

### Arguments

#### Required

* `url` - this is the path to your Conjur instance endpoint.  e.g. `https://conjur.cyberark.com:8443`
* `account` - this is the account configured for the Conjur instance during deployment.
* `host_id` - this is the Host ID granted to your application by Conjur when created via policy. e.g. `host/db/github_action`
* `api_key` - this is the API key associated with your Host ID declared previously.
* `secrets` - a semi-colon delimited list of secrets to fetch.  Refer to [Secrets Syntax](#secrets-syntax) in the README below for more details.

#### Optional

* `certificate` - if using a self-signed certificate, provide the contents for validated SSL.
* `authn_token_file` - this is the file path for the Conjur auth token.

#### Not required
* `authn_id` - this is the ID of Authn-JWT at Conjur
