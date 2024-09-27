########################

To make the automated deployment install the ansible by using following commands:

sudo apt update

sudo apt install ansible

ansible --version

After installing the ansible we are ready to apply the playbook,
RUN the playbook by using following command:

ansible-playbook -i inventory.yaml eks_deployment_playbook.yml
