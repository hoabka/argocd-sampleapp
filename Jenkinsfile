pipeline {
  options {
    timestamps()
    timeout(time: 180, unit: 'MINUTES')
    ansiColor('xterm')
    disableConcurrentBuilds()
    buildDiscarder(logRotator(numToKeepStr: '250', daysToKeepStr: '5'))
  }

  agent {
    dockerfile {
      additionalBuildArgs '${DOCKER_NO_CACHE} --tag "packer-demo/jenkins-agent:${IMG_LABEL}" --build-arg "TERRAFORM_VERSION=${TERRAFORM_VERSION}" --build-arg "PACKER_VERSION=${PACKER_VERSION}" --build-arg "USER_ID=$(user_id="$(id --user)"; [[ "${user_id}" == 0 ]] && echo "1000" || echo "${user_id}")" --build-arg "GROUP_ID=$(group_id="$(id --group)"; [[ "${group_id}" == 0 ]] && echo "1000" || echo "${group_id}")"'
    }
  }
  environment {
    AWS_ACCESS_KEY_ID     = credentials('PACKER_AWS_ACCESS_KEY')
    AWS_SECRET_ACCESS_KEY = credentials('PACKER_AWS_SECRET_KEY')
    WORK_SUB_DIR          = 'vm-based/infra/aws'
    JENKINS_USER_ID       = 'user'
    JENKINS_API_TOKEN     = credentials('JENKIN_API_TOKEN')
  }

  parameters {
    choice(
      choices: ['deployment' , 'cleanup'],
      description: '',
      name: 'REQUESTED_ACTION'
    )
    string(
      name: 'IMG_LABEL',
      defaultValue: 'latest',
      description: 'docker container image label name'
    )
    choice(
      name: 'TF_REMOTE_USER',
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
      when {
                // Deployment
                expression { params.REQUESTED_ACTION == 'deployment' }
      }
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

    stage('init-deployment') {
    when {
                // Deployment
                expression { params.REQUESTED_ACTION == 'deployment' }
      }
      options {
        timeout(time: 10, unit: 'MINUTES')
      }

      steps {
        sh '''#!/usr/bin/env bash
          echo "Shell Process ID: $$"
          cd "${WORK_SUB_DIR}"
          bash scripts/init-deployment.bash
        '''
      }
    }

    stage('cleanup') {
    when {
         // Cleanup
         expression { params.REQUESTED_ACTION == 'cleanup' }
      }
      options {
        timeout(time: 10, unit: 'MINUTES')
      }

      steps {
        sh '''#!/usr/bin/env bash
          echo "Shell Process ID: $$"
          cd "${WORK_SUB_DIR}"
          sed -i 's/\r$//' scripts/*
          bash scripts/post-cleanup.bash
        '''
      }
    }
  }

  post {
    failure {
        sh '''#!/usr/bin/env bash
          echo "Shell Process ID: $$"
          cd "${WORK_SUB_DIR}"
          bash scripts/post-cleanup.bash
        '''
        }
    }
}
