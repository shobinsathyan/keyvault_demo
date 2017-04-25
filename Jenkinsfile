#!/usr/bin/env groovy

  stage('Retrieve') {
    node {
      deleteDir()
      checkout scm
      sh 'az keyvault secret show --vault-name \'JenkinsVault\' --name \'ContainerKey\' | grep -i value | awk \'{print $2}\' | sed  \'s/\\"//g\' > .ContainerKey'
      sh 'az keyvault secret show --vault-name \'JenkinsVault\' --name \'ContainerName\' | grep -i value | awk \'{print $2}\' | sed  \'s/\\"//g\' > .ContainerName'
      container_key = readFile '.ContainerKey'
      container_name = readFile '.ContainerName'
    }
  }

  stage('Compile') {
    node {
      sh "chmod 400 ./keys/key.pem"
      }
  }

  stage('Backups') {
    node {
      git branch: 'master', credentialsId: 'ce0ef50a-900b-4578-bac9-00357a6bbd63', url: 'git@github.com:shobinsathyan/keyvault_demo.git'
      sh 'chmod 400 ./keys/key.pem;export ANSIBLE_ROLES_PATH="../roles"'
      sh 'export ANSIBLE_CONFIG="ansible.cfg"'
      sh """cat <<EOF > ./new.ini
[azure]
tags=uuid:${env.UUID}
EOF"""
      sh "export ANSIBLE_HOST_KEY_CHECKING=False; export AZURE_INI_PATH=./new.ini; ansible-playbook -i ./inventory/azure_rm.py --vault-password-file /var/lib/jenkins/vault-password -u mpadmin --private-key=.keys/key.pem   --extra-vars='container_name=${container_name} container_key=${container_key}' --tags=runbackup master.yml"
    }
}
