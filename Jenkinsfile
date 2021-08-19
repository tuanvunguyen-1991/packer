pipeline {
  options {
    timestamps()
    timeout(time: 180, unit: 'MINUTES')
    ansiColor('xterm')
    disableConcurrentBuilds()
    buildDiscarder(logRotator(numToKeepStr: '250', daysToKeepStr: '5'))
  }

  agent any
  environment {
    AWS_ACCESS_KEY_ID     = credentials('PACKER_AWS_ACCESS_KEY')
    AWS_SECRET_ACCESS_KEY = credentials('PACKER_AWS_SECRET_KEY')
    WORK_SUB_DIR          = 'vm-based/infra/packer'
    JENKINS_USER_ID       = 'user'
    JENKINS_API_TOKEN     = credentials('JENKIN_API_TOKEN')
  }

  parameters {
    string(
      name: 'IMG_LABEL',
      defaultValue: 'latest',
      description: 'docker container image label name'
    )
    choice(
      name: 'PACKER_REMOTE_USER',
      choices: [
        'ubuntu',
      ],
      description: 'packer remote user'
    )
    choice(
      name: 'TERRAFORM_VERSION',
      choices: [
        '0.12.24',
      ],
      description: 'terraform version'
    )
    choice(
      name: 'PACKER_VERSION',
      choices: [
        '1.5.6',
      ],
      description: 'packer version'
    )
  }

  stages {
    stage('terraform-apply') {
      options {
        timeout(time: 10, unit: 'MINUTES')
      }

      steps {
        sh '''#!/usr/bin/env bash
          echo "Shell Process ID: $$"
          cd "${WORK_SUB_DIR}"
          sed -i 's/\r$//' scripts/*
          bash scripts/tf-apply.bash
        '''
      }
    }

    stage('packaging-image') {
      options {
        timeout(time: 150, unit: 'MINUTES')
      }

      steps {
        sh '''#!/usr/bin/env bash
          echo "Shell Process ID: $$"
          set -o errexit
          readonly ROLE_LIST=(
            web
            app
          )
          cd "${WORK_SUB_DIR}"
          for role_name in "${ROLE_LIST[@]}"
          do
            bash scripts/packer-build.sh $role_name
          done
        '''
      }
    }
  }

  post {
    cleanup {
      sh '''#!/usr/bin/env bash
        echo "Shell Process ID: $$"
        cd "${WORK_SUB_DIR}"
        bash scripts/post-cleanup.sh
      '''
    }
  }
}
