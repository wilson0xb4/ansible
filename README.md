# Ansible Deployment for Strava Friend Breaker

### Assumptions
- `sudo pip install ansible boto six`
- `ansible-galaxy install jdauphant.nginx`
- AWS Credentials are exported to the local environment
    - (and have the ability to create/terminate EC2 instances, security groups, load balancers, AMIs, auto-scaling groups)
    - `export AWS_ACCESS_KEY_ID=""`
    - `export AWS_SECRET_ACCESS_KEY=""`
    - set `ssh-key` in `group_vars/all/vars.yml` to the name of a ssh-key connected with your AWS account
- put `sfb_deploy` in `files/` directory (ssh key only used for github deployment)
- put `vault` in `group_vars/all/` directory (random api keys)

- `ansible-playbook -i inventory/ site.yml` will:
    - setup a loadbalancer
    - provision an EC2 instance
    - deploy django, gunicorn, celery, nginx
    - create an AMI of that instance
    - terminate the temporary instance
    - create a launch configuration
    - and finally create the auto-scaling group
    - site should be viewable at the reported address


### Other Notes...
I created an IAM user for SQS and used those credentials on the deployments. This shouldn't matter currently as I think the load balancer setup broke the celery/sqs interaction.
