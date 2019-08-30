# devAnsibleCF
Ansible CF

### Notes
Cloudformation features change often, and this module tries to keep up. That means your botocore version should be fresh. The version listed in the requirements is the oldest version that works with the module as a whole. Some features may require recent versions, and we do not pinpoint a minimum version for each feature. Instead of relying on the minimum version, keep botocore up to date. AWS is always releasing features and fixing bugs.

If parameters are not set within the module, the following environment variables can be used in decreasing order of precedence AWS_URL or EC2_URL, AWS_ACCESS_KEY_ID or AWS_ACCESS_KEY or EC2_ACCESS_KEY, AWS_SECRET_ACCESS_KEY or AWS_SECRET_KEY or EC2_SECRET_KEY, AWS_SECURITY_TOKEN or EC2_SECURITY_TOKEN, AWS_REGION or EC2_REGION

Ansible uses the boto configuration file (typically ~/.boto) if no credentials are provided. See https://boto.readthedocs.io/en/latest/boto_config_tut.html

AWS_REGION or EC2_REGION can be typically be used to specify the AWS region, when required, but this can also be configured in the boto config file

## Variables

### Overview

The following variables are available in `defaults/main.yml` and can be used to setup your infrastructure.

| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `cloudformation_clean_build_env` | bool | `False` | Clean `build/` directory of Jinja2 rendered Cloudformation templates on each run. |
| `cloudformation_generate_only` | bool | `False` | Insteaf of deploying your Cloudformation templates, you can also only render them and have them available in the `build/` directory so you can use your current infrastructure to deploy those templates.<br/>**Hint:** Specify this variable via ansible command line arguments |
| `cloudformation_run_diff` | bool | `False` | This role ships a custom Ansible Cloudformation module **[cloudformation_diff](https://github.com/cytopia/ansible-modules)**. This module generates a text-based diff output between your local cloudformation template ready to be deployed and the currently deployed templated on AWS Cloudformation.<br/>Why would I want this?<br/>The current cloudformation module only list change sets in --check mode, which will let you know what *kind* will change (e.g. security groups), but not what exactly will change (which security groups and the values of them) In order to also be able to view the exact changes that will take place, enable the cloudformation_diff module here. |
| `cloudformation_diff_output` | string | `json` | When `cloudformation_run_diff` is enabled, what output diff should be specified? If you write your cloudformation templates via json, use `json` here or if you write your cloudformation templates in yaml, use `yaml` here. |
| `cloudformation_required` | list | `[]` | Array of available cloudformation stack keys that you want to enforce to be required instead of being optional. Each cloudformation stack item will be checked against the customly set required keys. In case a stack item does not contain any of those keys, an error will be thrown before any deployment has happened. |
| `cloudformation_defaults` | dict | `{}` | Dictionary of default values to apply to every cloudformation stack. Note that those values can still be overwritten on a per stack definition. |
| `cloudformation_stacks` | list | `[]` | Array of cloudformation stacks to deploy. |

### Details

This section contains a more detailed describtion about available dict or array keys.

#### `cloudformation_defaults`

| Key | Type | Required | Description |
|-----|------|----------|-------------|
| `aws_access_key` | string | optional | AWS access key to use |
| `aws_secret_key` | string | optional | AWS secret key to use |
| `security_token` | string | optional | AWS security token to use |
| `profile` | string | optional | AWS boto profile to use |
| `notification_arns` | string | optional | Publish stack notifications to these ARN's |
| `termination_protection` | bool | optional | Enable or disable termination protection on the stack. Only works with botocore >= 1.7.18 |
| `region` | string | optional | AWS region to deploy stack to |

#### `cloudformation_stacks`

| Key | Type | Required | Description |
|-----|------|----------|-------------|
| `stack_name` | string | required | Name of the cloudformation stack |
| `template` | string | required | Path to the cloudformation template to render and deploy (Does not need to be rendered) |
| `aws_access_key` | string | optional | AWS access key to use (overwrites default) |
| `aws_secret_key` | string | optional | AWS access key to use (overwrites default)  |
| `security_token` | string | optional | AWS security token to use (overwrites default) |
| `profile` | string | optional | AWS boto profile to use (overwrites default) |
| `notification_arns` | string | optional | Publish stack notifications to these ARN's (overwrites default) |
| `termination_protection` | bool | optional | Enable or disable termination protection on the stack. Only works with botocore >= 1.7.18 |
| `region` | string | optional | AWS region to deploy stack to (overwrites default) |
| `template_parameters` | dict | optional | Required cloudformation stack parameters |
| `tags` | dict | optional | Tags associated with the cloudformation stack |

### Examples

Define default values to be applied to all stacks (if not overwritten on a per stack definition)
```yml
# Enforce that 'profile' must be set for each cloudformation stack item
cloudformation_required:
  - profile

cloudformation_defaults:
  region: eu-central-1
```


Define cloudformation stacks to be rendered and deployed
```yml
cloudformation_stacks:
  - stack_name: stack-s3
    template: files/cloudformation/s3.yml.j2
    profile: production
    template_parameters:
      bucketName: my-bucket
    tags:
      env: production
  - stack_name: stack-lambda
    template: files/cloudformation/lambda.yml.j2
    profile: production
    termination_protection: True
    template_parameters:
      lambdaFunctionName: lambda
      handler: lambda.run_handler
      runtime: python2.7
      s3Bucket: my-bucket
      s3Key: lambda.py.zip
    tags:
      env: production
```

Only render your Jinja2 templates, but do not deploy them to AWS. Rendered cloudformation files will be inside the `build/` directory of this role.
```bash
$ ansible-playbook play.yml -e cloudformation_generate_only=True
```

## Usage

### Simple

Basisc usage example:

`playbook.yml`
```yml
- hosts: localhost
  connection: local
  roles:
    - cloudformation
```

`group_vars/all.yml`
```yml
# Define Cloudformation stacks
cloudformation_stacks:
  # First stack
  - stack_name: stack-s3
    profile: testing
    region: eu-central-1
    template: files/cloudformation/s3.yml.j2
    template_parameters:
      bucketName: my-bucket
    tags:
      env: testing
  # Second stack
  - stack_name: stack-lambda
    profile: testing
    termination_protection: True
    region: eu-central-1
    template: files/cloudformation/lambda.yml.j2
    template_parameters:
      lambdaFunctionName: lambda
      handler: lambda.run_handler
      runtime: python2.7
      s3Bucket: my-bucket
      s3Key: lambda.py.zip
    tags:
      env: testing
```

### Advanced

Advanced usage example calling the role independently in different *virtual* hosts.

`inventory`
```ini
[my-group]
infrastructure  ansible_connection=local
application     ansible_connection=local
```

`playbook.yml`
```yml
# Infrastructure part
- hosts: infrastructure
  roles:
    - cloudformation
  tags:
    - infrastructure

# Application part
- hosts: application
  roles:
    - some-role
  tags:
    - some-role
    - application

- hosts: application
  roles:
    - cloudformation
  tags:
    - application
```

`group_vars/my-group.yml`
```yml
stack_prefix: testing
boto_profile: testing
s3_bucket: awesome-lambda

cloudformation_defaults:
  profile: "{{ boto_profile }}"
  region: eu-central-1
```

`host_vars/infrastructure.yml`
```yml
cloudformation_stacks:
  - stack_name: "{{ stack_prefix }}-s3"
    template: files/cloudformation/s3.yml.j2
    template_parameters:
      bucketName: "{{ s3_bucket }}"
    tags:
      env: "{{ stack_prefix }}"
```

`host_vars/application.yml`
```yml
cloudformation_stacks:
  - stack_name: "{{ stack_prefix }}-lambda"
    template: files/cloudformation/lambda.yml.j2
    template_parameters:
      lambdaFunctionName: lambda
      handler: lambda.run_handler
      runtime: python2.7
      s3Bucket: "{{ s3_bucket }}"
      s3Key: lambda.py.zip
    tags:
      env: "{{ stack_prefix }}"
```


## Templates

This section gives a brief overview about what can be done with Cloudformation templates using Jinja2 directives.

### Example: Subnet definitions

The following template can be rolled out to different staging environment and is able to include a different number of subnets.

Ansible variables
```yml
---
# file: staging.yml
vpc_subnets:
  - directive: subnetA
    az: a
    cidr: 10.0.10.0/24
    tags:
      - name: Name
        value: staging-subnet-a
      - name: env
        value: staging
  - directive: subnetB
    az: b
    cidr: 10.0.20.0/24
    tags:
      - name: Name
        value: staging-subnet-b
      - name: env
        value: staging
```

```yml
---
# file: production.yml
vpc_subnets:
  - directive: subnetA
    az: a
    cidr: 10.0.10.0/24
    tags:
      - name: Name
        value: prod-subnet-a
      - name: env
        value: production
  - directive: subnetB
    az: b
    cidr: 10.0.20.0/24
    tags:
      - name: Name
        value: prod-subnet-b
      - name: env
        value: production
  - directive: subnetC
    az: b
    cidr: 10.0.30.0/24
    tags:
      - name: Name
        value: prod-subnet-c
      - name: env
        value: production
```

Cloudformation template
```jinja
AWSTemplateFormatVersion: '2010-09-09'
Description: VPC Template
Resources:
  vpc:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: {{ vpc_cidr_block }}
      EnableDnsSupport: true
      EnableDnsHostnames: true
{% if vpc_tags %}
      Tags:
{% for tag in vpc_tags %}
        - Key: {{ tag.name }}
          Value: {{ tag.value }}
{% endfor %}
{% endif %}
{% for subnet in vpc_subnets %}
  {{ subnet.directive }}:
    Type: AWS::EC2::Subnet
    Properties:
      AvailabilityZone: {{ subnet.az }}
      CidrBlock: {{ subnet.cidr }}
      VpcId: !Ref vpc
{% if subnet.tags %}
      Tags:
{% for tag in subnet.tags %}
        - Key: {{ tag.name }}
          Value: {{ tag.value }}
{% endfor %}
{% endif %}
```


### Example: Security groups

Defining security groups with IP-specific rules is very difficult when you want to keep environment agnosticity and still use the same Cloudformation template for all environments. This however can easily be overcome by providing environment specific array definitions via Jinja2.

Ansible variables
```yml
---
# file: staging.yml
# Staging is wiede open, so that developers are able to
# connect from attached VPN's
security_groups:
  - protocol:  tcp
    from_port: 3306
    to_port:   3306
    cidr_ip:   10.0.0.1/32
  - protocol:  tcp
    from_port: 3306
    to_port:   3306
    cidr_ip:   192.168.0.15/32
  - protocol:  tcp
    from_port: 3306
    to_port:   3306
    cidr_ip:   172.16.0.0/16
```

```yml
---
# file: production.yml
# The production environment has far less rules as well as other
# ip ranges.
security_groups:
  - protocol:  tcp
    from_port: 3306
    to_port:   3306
    cidr_ip:   10.0.15.1/32
```

Cloudformation template
```jinja
AWSTemplateFormatVersion: '2010-09-09'
Description: VPC Template
Resources:
  rdsSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: RDS security group
{% if security_groups %}
      SecurityGroupIngress:
{% for rule in security_groups %}
        - IpProtocol: "{{ rule.protocol }}"
          FromPort: "{{ rule.from_port }}"
          ToPort: "{{ rule.to_port }}"
          CidrIp: "{{ rule.cidr_ip }}"
{% endfor %}
{% endif %}
```

## Diff

When having enable `cloudformation_run_diff`, you will be able to see line by line diff output from you local (jinja2 rendered) template against the one which is currently deployed on AWS. To give you an impression about how this looks, see the following example output:

Make sure to run Ansible with `--diff` to make it work:
```bash
$ ansible-playbook play.yml --diff
```

### Json diff
To have it output in json diff mode, set `cloudformation_diff_output` to `json`.
```diff
TASK [cloudformation : diff cloudformation template file] *********************************************
--- before
+++ after
@@ -38,7 +38,6 @@
             "Type": "AWS::S3::BucketPolicy"
         },
         "s3Bucket": {
-            "DeletionPolicy": "Retain",
             "Properties": {
                 "BucketName": {
                     "Ref": "bucketName"
```

### Yaml diff
To have it output in yaml diff mode, set `cloudformation_diff_output` to `yaml`.
```diff
TASK [cloudformation : diff cloudformation template file] *********************************************
--- before
+++ after
@@ -14,7 +14,6 @@
               Service: !Sub 'logs.${AWS::Region}.amazonaws.com'
       Bucket: !Ref 's3Bucket'
   s3Bucket:
-    DeletionPolicy: Retain
     Type: AWS::S3::Bucket
     Properties:
       BucketName: !Ref 'bucketName'
```


## Dependencies

This role does not depend on any other roles.


## Requirements

Use at least **Ansible 2.5** in order to also have `--check` mode for cloudformation.

The python module `cfn_flip` is required, when using line-by-line diff of local and remote Cloudformation templates (`cloudformation_run_diff=True`). This can easily be installed locally:
```bash
$ pip install cfn_flip
```


## License

[MIT License](LICENSE.md)

Copyright (c) 2017 [cytopia](https://github.com/cytopia)
