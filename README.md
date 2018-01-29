# EC2 Hostname

Role set "incremental" tag:Name.


## Algorithm

1. gather all instances their tags

2. generate new tag:Name which isn't taken using one of provided rules (name, name prefix or asg)

3. set new tag:Name

4. sleep for random number of seconds (1-10)

5. gather all instances and check collisions

5.1 if collisions occurred go to step 2.

5.2 if no collisions found then stop

This role should be applied 60 seconds after instance boot, because EC2 instance tags are unavailable while instance in `pending` state (generally 10-20 seconds after instance launch). This role can be executed locally, for example, using cloud-init via user data.

## Usage

```yaml
---
- hosts: localhost
  connection: local
  become: yes
  roles:
    - ansible-role-ec2-hostname
```

## Vars

* `ec2_hostname_name` [default: ``]: name that should be set.
* `ec2_hostname_name_prefix` [default: ``]: prefix that should be used for new name generation.
* `ec2_hostname_name_prefix_asg` [default: `False`]: flag that tell to get asg group name and use it as a prefix
* `ec2_hostname_retries` [default: `10`]: how many times script should try to set name in case of collision.
* `ec2_hostname_overwrite` [default: `no`]: should existing name be overwritten.
* `ec2_hostname_verbose` [default: `yes`]: informative logging.
* `ec2_hostname_debug` [default: `no`]: debug logging.
* `ec2_hostname_linux_hostname` [default: `yes`]: set hostname on target instances.


## IAM

Following IAM policy should be attached to instance profile role.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:CreateTags",
                "ec2:DescribeTags",
                "ec2:DescribeInstances"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```
