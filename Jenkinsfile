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
    WORK_SUB_DIR          = 'container-based'
    REGISTRY              = '633834615594.dkr.ecr.us-west-2.amazonaws.com'
  }

  parameters {
    string(
      name: 'artifactURL',
      defaultValue: 'https://github.com/AKSarav/SampleWebApp/raw/master/dist/SampleWebApp.war',
      description: 'Application artifact URL'
    )
  }

  stages {
    stage('docker-build') {
      options {
        timeout(time: 10, unit: 'MINUTES')
      }

      steps {
        sh '''#!/usr/bin/env bash
          echo "Shell Process ID: $$"
          cd "${WORK_SUB_DIR}"
          docker build --tag "samplewebapp:${BUILD_NUMBER}" --build-arg "artifactURL=${artifactURL}" .
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
          cd "${WORK_SUB_DIR}"
          region=`echo $REGISTRY | awk -F '.' '{print $4}'`
          aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${REGISTRY}
          docker tag samplewebapp:${BUILD_NUMBER} "${REGISTRY}/samplewebapp:${BUILD_NUMBER}"
          docker push "${REGISTRY}/samplewebapp:${BUILD_NUMBER}"
          echo "IMAGE_NAME=${REGISTRY}/samplewebapp:${BUILD_NUMBER}" > build.properties
        '''
      }
    }
    stage('Archive properties') {
      options {
        timeout(time: 10, unit: 'MINUTES')
      }

      steps {
        archiveArtifacts artifacts: 'container-based/build.properties'
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
