# devAnsibleCF
Ansible CF

### Notes
Cloudformation features change often, and this module tries to keep up. That means your botocore version should be fresh. The version listed in the requirements is the oldest version that works with the module as a whole. Some features may require recent versions, and we do not pinpoint a minimum version for each feature. Instead of relying on the minimum version, keep botocore up to date. AWS is always releasing features and fixing bugs.

If parameters are not set within the module, the following environment variables can be used in decreasing order of precedence AWS_URL or EC2_URL, AWS_ACCESS_KEY_ID or AWS_ACCESS_KEY or EC2_ACCESS_KEY, AWS_SECRET_ACCESS_KEY or AWS_SECRET_KEY or EC2_SECRET_KEY, AWS_SECURITY_TOKEN or EC2_SECURITY_TOKEN, AWS_REGION or EC2_REGION

Ansible uses the boto configuration file (typically ~/.boto) if no credentials are provided. See https://boto.readthedocs.io/en/latest/boto_config_tut.html

AWS_REGION or EC2_REGION can be typically be used to specify the AWS region, when required, but this can also be configured in the boto config file

