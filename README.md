# ansible-w3cunicorn

Template for deployment of the w3c-Unicorn validation tool, ideally for use in a CI infrastructure

The Ansible playbook defined under "deploy" does the following:

- Provision nodes in Vagrant or AWS
    - Provisions nodes
    - Setups DNS (including hosts for local dev)
    - Provisions VPC and security groups
- Deploy and configure w3c-unicorn
    - deploys all packages
    - builds w3c-unicorn
    - configures tomcat7

w3c-Unicorn is accessible in the vagrant setup at http://validator:8080/unicorn following deploy.

### Requirements

- ansible
- vagrant
- aws

Everything else is deployed for you.

### Parameters
For each environment definition is done in the '/deploy/group_vars/', for example local using vagrant:

        ansible_ssh_user: vagrant
        remote_user: vagrant
        local_sudo_user: root

        private_key_file: ~/.vagrant.d/insecure_private_key

        zone: local.domain

        # Application Git repo variables
        # only enable git_repo or git_local_repo
        #
        git_repo_local: /project_repo
        git_branch: HEAD

        # provider info
        provider: vagrant

        tomcat_user: admin
        tomcat_password: s3cret

        vagrant_nodes:
          - name: validator
            groups:
              - w3cunicorn
            ip: '192.168.3.90'
            http: 9080
            ssl: 9443

This will result in three test nodes being provisioned in vagrant and then the Ansible playbooks for each group bing applied.
For AWS  you will need to put in your own secrets for provisioning to work:

        # AWS keys

        ec2_access_key: "your key"
        ec2_secret_key: "your key"


### Workflow

#### Create an environment (Provision and Deploy)

    ansible-playbook -i inventory/stubinv w3cdeploy.yml --ask-sudo-pass

Now I do need to start off with an quirk here, the first time you spin up a vagrant environment the inventory is not
going to exist so I placed a stub inventory into the project. This will only execute the provision portion.
On all subsequent executions the linked file 'w3c-ansible' will exists, which links to the generated inventory from the vagrant provision.
Hence the following command is used to execute the Provision and Deploy, even if you destroy your vagrant nodes:

    ansible-playbook -i inventory/w3c-ansible w3cdeploy.yml --ask-sudo-pass

The `--ask-sudo-pass`, will prompt for your local sudo password so that '/etc/hosts' can be updated with the node names. This is only needed for vagrant environments.


#### Deploy updated code to an existing environment

Local vagrant or AWC environment:

    ansible-playbook -i inventory/w3c-ansible w3cunicorn.yml

