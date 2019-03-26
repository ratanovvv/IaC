# AWS TEST TASK
## Task itself
1. EC2 instances count is configurable;
2. ELB listeners protocol - TCP, HTTPS is handeled by EC2 instances, not ELB;
3. * If there is N instances already and you start playbook with N+M, then M EC2 instances will be created, configured and attached to ELB.
   * Configuration, nginx version and etc will be updated according to playbook on N already running instances;
4. Nginx is showing page with EC2 instance aws id and time of creation;
5. It should be HTTPS;
6. * Letsencrypt cert (equal on every EC2 instance) is being updated on every instance with every start of playbook ("renew" in letsencrypt vocabulary).
   * No new cert creation, old one update;
7. Initial cert issuing is managed by another playbook, domain owner right is validated through DNS challenge (route 53). No credentials on instances.
## Versions
```bash
ansible --version
ansible 2.6.2
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/home/ec2-user/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/site-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.14 (default, Mar 16 2018, 18:20:07) [GCC 7.2.1 20170915 (Red Hat 7.2.1-2)]
```
## Prerequisite
Create EC2 ssh key in aws cloud and define label in `group_vars/all.yml`
## Checkout
Get code from [Git repo](https://github.com/ratanovvv/ansible.git)
```bash
git clone --single-branch --branch aws-ansible https://github.com/ratanovvv/IaC.git
```
## Variables
Define in `group_vars/all.yml`
## Secrets
- Encrypt `group_vars/all.yml` with vault
- Encrypt private ssh key with vault
## Create IaaS
`ansible-playbook -i inventory/hosts aws.yml --tags "iaas_only" --ask-vault`
should fail on `ec2_dyn_group [GATHERING FACTS]`
## Prepare vms
aws ubuntu image is without python inside. Install it
`ssh -i /path/to/ssh/key ubuntu@PUBLIC_IP "sudo apt-get update && sudo apt-get install -y python"`
## Choose vm for issuing certificate
edit `letsencrypt_issuer` group in inventories/hosts
## Create certificate
`ansible-playbook -i inventory/hosts aws.yml --tags "init" --ask-vault`
## Renew certificate and nginx
`ansible-playbook -i inventory/hosts aws.yml --tags "update" --ask-vault`