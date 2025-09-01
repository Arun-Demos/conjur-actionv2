Conjur → AWS Env (Parser Only)

This GitHub Action parses a Conjur dynamic secret JSON (with AWS credentials under .data.*) and exports them as standard AWS environment variables.

It does not run aws-actions/configure-aws-credentials — developers can choose how to configure AWS (e.g. add proxy settings, assume role, custom session names, etc.).

📦 Inputs
Name	Required	Default	Description
json	No	""	JSON string containing the Conjur dynamic secret. If omitted, the action will read from the environment variable defined by json_env_name.
json_env_name	No	CONJUR_AWS_JSON	Name of the environment variable that holds the JSON. Useful when pairing with the Conjur fetch action.
region	No	""	AWS region to set as AWS_DEFAULT_REGION.
mask_full_json	No	"true"	Whether to mask the entire JSON string in logs.
📤 Outputs
Name	Description
access_key_id	Parsed AWS access key ID (masked in logs).
assumed_role_user_arn	ARN of the assumed role (from the JSON, if present).
🛠️ Example: Fetch + Parse + Configure
permissions:
  id-token: write
  contents: read

jobs:
  s3-list:
    runs-on: self-hosted
    steps:
      - uses: actions/checkout@v4

      # Step 1: Fetch JSON from Conjur
      - name: Fetch AWS creds JSON from Conjur
        uses: your-org/conjur-actions/fetch@v1
        with:
          url: https://<tenant>.secretsmgr.cyberark.cloud/api
          account: conjur
          authn_id: <YourServiceId>
          secrets: |
            data/aws/dynamic/creds|CONJUR_AWS_JSON

      # Step 2: Parse JSON → AWS env
      - name: Parse Conjur JSON
        uses: your-org/conjur-actions/aws-from-conjur-json@v1
        with:
          region: ap-southeast-1

      # Step 3: Developer decides how to configure AWS (e.g. with proxy)
      - name: Configure AWS
        env:
          HTTPS_PROXY: http://proxy.corp:8080
          NO_PROXY: .amazonaws.com,.cyberark.cloud
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id:     ${{ env.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ env.AWS_SECRET_ACCESS_KEY }}
          aws-session-token:     ${{ env.AWS_SESSION_TOKEN }}
          aws-region:            ${{ env.AWS_DEFAULT_REGION }}

      - run: aws s3 ls

🛠️ Example: Direct JSON (no fetch step)
- name: Parse JSON to AWS env
  uses: your-org/conjur-actions/aws-from-conjur-json@v1
  with:
    json: ${{ secrets.CONJUR_AWS_JSON }}
    region: us-east-1

- run: aws s3 ls

🔐 Security

All secrets (access_key_id, secret_access_key, session_token) are masked in logs.

Only non-sensitive outputs (assumed_role_user_arn) are exposed.
