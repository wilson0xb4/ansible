# Ansible Deployment for Strava Friend Breaker

### Assumptions
- `sudo pip install ansible boto six`
- `ansible-galaxy install jdauphant.nginx`
- AWS Credentials are exported to the local environment
    - (and have the ability to create/terminate EC2 instances, security groups, load balancers, AMIs, auto-scaling groups)
    - `export AWS_ACCESS_KEY_ID=""`
    - `export AWS_SECRET_ACCESS_KEY=""`
    - set `ssh-key` in `group_vars/all/vars.yml` to the name of a ssh-key connected with your AWS account
- put `sfb_deploy` in `files/` directory (its an ssh key only used for github deployment)
- put `vault.yml` in `group_vars/all/` directory (contains random api keys)

- `ansible-playbook -i inventory/ site.yml` will:
    - create a security group   
    - setup a loadbalancer
    - provision an EC2 instance
    - deploy django, gunicorn, celery, nginx
    - create an AMI of that instance
    - terminate the temporary instance
    - create a launch configuration
    - and finally create the auto-scaling group
    - site should be viewable at the reported address

### Bonus 1: Dynamic DNS
Executing `inventory/ec2.py` grabs the dynamic inventory from AWS and provides lots of info.

### Bonus 2: Monitoring and Alerting
I didn't get to implement this, but I was looking at using [ELK stack](https://www.elastic.co/webinars/elk-stack-devops-environment).
From their website:
- Elasticsearch for deep search and data analytics
- Logstash for centralized logging, log enrichment and parsing
- Kibana for powerful and beautiful data visualizations

### Bonus 3: Scalability
I went with an immutable infrastructure design. There is a load balancer that is the entry point to the site, all instances are deployed under it. When a new deployment happens, we provision a temporary ec2 instance, deploy and configure the app, create an AMI, and destroy the temporary instance. Then the auto-scaling policy starts provisioning new instances from the AMI we created under the load balancer, while also removing the old instances from it. It does this at a specified rate, I have it set to remove one at a time.

Currently the only metric its basing this off of is that `x` instances are 'healthy'. If I can get my celery/sqs system back up, I want to make the metric based on that. Looking at how full the queues get maybe.

### Other Notes...
I created an IAM user for SQS and used those credentials on the deployments. This shouldn't matter currently as I think the load balancer setup broke the celery/sqs interaction.
