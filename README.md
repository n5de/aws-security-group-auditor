## AWS EC2 Security Group Auditor

The purpose of the script is to compare the currently running AWS EC2 security groups with your local, audited and reviewed security groups.
If the scrip finds any missing or extra rules, it will note the missing ore extra rules.

### Options

* `--dump-ec2` - Rather than performing the check, dump the JSON representation of our security groups
* `--json-diff` - Print the changed rules as JSON instead of English
* `--nagios-output` - Return nagios compatible check result outputs
* `--aws-conf`/`-a` - Path to the AWS config file.
* `--rule_root`/`-r` - Path to the local rules root.
* `--rule_path`/`-d` - Path to the local rules.
* `--region` - AWS region to query

### AWS credentials

For now the script works from AWS CLI compatible ini style config. By default it looks for `~/.aws/credentials`. I'm plan to add `STS`-`AssumedRole` support too. Stay tuned.

The credentials file should look like below:

```bash
[profile-name]
aws_access_key_id     = AKIA...
aws_secret_access_key = Ze7A...
```

### Directory structure

Each security group is a separate `JSON` file under `RULE_ROOT`/`AWS_OWNER_ID`/ `AWS_REGION` directory.

### Security Group JSON files

The naming convention of a rule file (if it's in a VPC) looks like:

`<security-group-name>_<security-group-id>_<vpc-id>.json`

If it's not inside of the VPC:

`<security-group-name>_<security-group-id>.json`

Once you dumped the rules, you can edit them and add comments.

Lines started with `#` are considered as comments.
JSON elemnts called `comments` are considered as commenst too, so feel free to use them.

Note that if you dump the rules again, you'll overwrite them, so try to dump only once and maintain your rules on a regular basis.

Example for structure and comments:

```json
{
     # Our shiny web server
     "123456789012:web-server:sg-8b8aa991:None": {
        "rules:-1--1-icmp": {
            "comment": "allow our nagios to ping this server",
            "from_port": "-1", "to_port": "-1", "ip_protocol": "icmp",
            "grants": [
                "123456789012:nagios:sg-9cede390"
            ]
        },
        "rules:22-22-tcp": {
            "comment": "Allow SSH from the the VPN only",
            "from_port": "22", "to_port": "22", "ip_protocol": "tcp",
            "grants": [
                "123456789012:openvpn:sg-44417aad"
            ]
        },
        "rules:443-443-tcp": {
            "comment": "Allow HTTPS from everywhere",
            "from_port": "443", "to_port": "443", "ip_protocol": "tcp",
            "grants": [ "0.0.0.0/0" ]
        },
        "rules:80-80-tcp": {
            "comment": "Allow HTTP from everywhere",
            "from_port": "80", "to_port": "80", "ip_protocol": "tcp",
            "grants": [ "0.0.0.0/0" ]
        }
    }
 }
```

### Installation

```bash
virtualenv virtualenv
. virtualenv/bin/activate
pip install -r requirements.txt
```

### Examples

Dump current Amazon EC2 us-east-1 security groups to `company_rules` directory using the `company-test` AWS credentials profile.

```bash
$ ./aws_sg_auditor --dump-ec2 --region us-east-1 --aws-profile company-test --rule_root company_rules

```

Compare the same local rules with the current EC2 rules

```bash
 ./aws_sg_auditor --region us-east-1 --aws-profile cloudbees-test \
 --rule_path company_rules/123456789012/us-east-1

```


### Credits

Special thanks to [Parse, Inc](http://parse.com) for the initial idea. I forked and rewrote their script. The original script - which is a nagios executable - can be found [here](https://github.com/ParsePlatform/Ops/tree/master/tools). 


