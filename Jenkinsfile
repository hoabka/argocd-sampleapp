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
    REGISTRY              = '633834615594.dkr.ecr.us-west-2.amazonaws.com'
    GIT_CREDS             = credentials('git')
  }

  stages {
    stage('docker-build') {
      options {
        timeout(time: 10, unit: 'MINUTES')
      }

      steps {
        sh '''#!/usr/bin/env bash
          echo "Shell Process ID: $$"
          docker build --tag "samplewebapp:${env.GIT_COMMIT}" .
        '''
      }
    }

    stage('docker-push-registry') {
      options {
        timeout(time: 10, unit: 'MINUTES')
      }

      steps {
        sh '''#!/usr/bin/env bash
          echo "Shell Process ID: $$"
          region=`echo $REGISTRY | awk -F '.' '{print $4}'`
          aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${REGISTRY}
          docker tag 'samplewebapp:${env.GIT_COMMIT}' '${REGISTRY}/samplewebapp:${env.GIT_COMMIT}'
          docker push '${REGISTRY}/samplewebapp:${env.GIT_COMMIT}'
        '''
      }
    }
     stage('Deploy DEV') {
      steps {
        sh '''#!/usr/bin/env bash
          echo "Shell Process ID: $$"
          git clone https://$GIT_CREDS_USR:$GIT_CREDS_PSW@github.com/hoabka/argocd-k8s-manifest.git
          git config --global user.email 'ci@ci.com'
          cd ./dev && kustomize edit set image '${REGISTRY}/samplewebapp:${env.GIT_COMMIT}'
          git commit -am 'Publish new version' && git push || echo 'no changes'
        '''
      }
    }
     stage('Deploy PROD') {
      steps {
        input message:'Approve deployment?'
        sh '''#!/usr/bin/env bash
          cd ./prod && kustomize edit set image '${REGISTRY}/samplewebapp:${env.GIT_COMMIT}'
          git commit -am 'Publish new version' && git push || echo 'no changes'
        '''
      }
    }
  }
  post {
    failure {
        sh '''#!/usr/bin/env bash
          echo "Shell Process ID: $$"
        '''
        }
    }
}