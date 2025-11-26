# Ansible Playbooks - pve-cloud-awx-cron

This repository contains the tls_renew playbook which scans the postgres for created certificates / configs and renews them with the given pve cloud acme method.

## Setup AWX

First you need to configure some prerequisites for AWX to be able to run the playbook.

1. Add the pve-cloud-ee as an execution environment
2. Add source control credential to git server with this repository
3. Add this repository as a project
4. Create job template for the tls_renew playbook, use this script to fetch secrets and set vars in job template:
```bash
# fill these
PVE_CLUSTER=
PVE_CLOUD_DOMAIN=
eval "$(pvclu get-online-host --target-pve $PVE_CLUSTER.$PVE_CLOUD_DOMAIN)"

# echo secrets

# set variable acme_accountkey with the result, use acme_accountkey: | and tab indent paste the key in the next line
ssh root@$PVE_ANSIBLE_HOST cat /etc/pve/cloud/secrets/acme-account.key

# set variable pg_conn_str with this echo result, put result in ''
echo "postgresql+psycopg2://postgres:$(ssh root@$PVE_ANSIBLE_HOST cat /etc/pve/cloud/secrets/patroni.pass)@$(ssh root@$PVE_ANSIBLE_HOST cat /etc/pve/cloud/cluster_vars.yaml | yq '.pve_haproxy_floating_ip_internal'):5000/pve_cloud?sslmode=disable"

# set acme_contact value with this, also put in ''
ssh root@$PVE_ANSIBLE_HOST cat /etc/pve/cloud/cluster_vars.yaml | yq '.acme_contact'
```
5. create daily schedule and enjoy. In pve-cloud-tf-monitoring module there is the option to add awx job failures to alerts.

the cron job inside pve-cloud-tf-controller will take care of syncing the tls secrets on the kubernetes side.