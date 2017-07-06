#The Ansible role AWS EC2

Add automatic EC2 instance provisioning via Ansible which can store on s3
Note when the application runs on the instance, it can use the role’s temporary credentials to access the bucket-name-goes-here S3 bucket. AWS administrator doesn’t have to grant the developer permission to access the bucket-name-goes-here bucket. The developer never has to share or manage credentials which is very risky in terms of security  compliance.

##Requirements

Requires Ansible and `boto` python package. You can install it via **PIP**
Example Playbook (aws_instance_store_on_s3.yml)

## Bucket Policy may look like:

{
"Version": "2017-07-06",
"Statement": [
{
"Effect": "Allow",
"Action": [
"s3:ListBucket"
],
"Resource": [
"arn:aws:s3:::s3-bucket-name-goes-here"
]
},
{
"Effect": "Allow",
"Action": [
"s3:PutObject",
"s3:GetObject",
"s3:DeleteObject",
"s3:ListObject"
],
"Resource": [
"arn:aws:s3:::s3-bucket-name-goes-here/*"
]
}
]
}


## list the items in the bucket after uploading to test it is working or not:

aws s3 ls s3://s3-bucket-name-goes-here

----------------

- name: Create an EC2 instance
ec2:
aws_access_key: "{{ aws_access_key }}"
aws_secret_key: "{{ aws_secret_key }}"
region: "{{ aws_region }}"
state: present
assign_public_ip: yes
group: '{{ security_group }}'
group_id: "{{ groups.group_id }}"
image: "{{ ec2.instance.ami_image }}"
instance_profile_name: iam_instance_role_shared_s3
instance_tags:
Env: "{{ env }}"
Name: "{{ prefix }}-{{ env }}-instance-01"
exact_count: 1
count_tag: 
Env: "{{ env }}"
Name: "{{ prefix }}-{{ env }}-instance-01"
instance_type: "{{ ec2.instance.type }}"
key_name: "{{ keyname }}"
private_ip: no
source_dest_check: true
tenancy: default
volumes:
- device_name: /dev/xvdb
device_type: standard
volume_size: 10
delete_on_termination: True
vpc_subnet_id: "{{ ec2.instance.subnets[0]['id'] }}"
wait: True
zone: "{{ ec2.instance.subnets[0]['az'] }}"
ec2_tag:
region: eu-west-1
resource: vol-name-id-goes-here
state: present
tags:
name: ec2
bucket: bucket-name-goes-here
register: ec2
