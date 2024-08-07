pipeline {
  agent any
  parameters {
      string(name: 'MAJOR_VERSION', defaultValue: '', description: 'Specify the major version (e.g., "1")')
      string(name: 'MINOR_VERSION', defaultValue: '', description: 'Specify the minor version (e.g., "2")')
  }

  environment {
    VERSION_FILE_NAME = "v.txt"
    PATCH_NUMBER = 0
    TAG = 0
    SSH_CREDENTIALS_ID = 'ssh_to_gitlab'
    GIT_URL = 'git@gitlab.com:<project_name>/<repo_name>.git'
    PERSONAL_ACCESS_TOKEN_ID = 'Personal_GitLab_Access_Token'
    REGISTRY_URL = 'registry.gitlab.com'
    REGISTRY_USERNAME = 'eliyahulevinson@gmail.com'
    REGISTRY_NAME = 'Ilon Mask'
    REGISTRY_PROJECT_URL = 'registry.gitlab.com/<project_name>/<repo_name>'
    EC2_COWSAY_URI = '<user>@<ec2_url>'
    COWSAY_IP = '44.204.135.123'
    COWSAY_PORT = '80'
  }

  stages {
      stage('Checkout') {
        steps {
          script {
            git branch: "${env.GIT_BRANCH}", credentialsId: "${env.SSH_CREDENTIALS_ID}", url: "${env.GIT_URL}"
          }
        }
      }

      stage('Build') {
        steps {
          script {
            sh "docker build -t cowsay:${env.IMAGE_TAG} ."
          }
        }
      }

    stage('Push') {
      steps {
        script {
          withCredentials(
            [string(credentialsId: "${env.PERSONAL_ACCESS_TOKEN_ID}", variable: 'TOKEN')]
          ) {
            sh """
              echo "${TOKEN}" | docker login "${env.REGISTRY_URL}" -u "${env.REGISTRY_USERNAME}" --password-stdin
              docker build -t "${env.REGISTRY_PROJECT_URL}:${env.IMAGE_TAG}" .
              docker push "${env.REGISTRY_PROJECT_URL}:${env.IMAGE_TAG}"
            """
          }
        }
      }
    }

    stage('Local Test') {
      steps {
        script {
          sh 'docker rm -f cowsay'
          sh "docker run -d --name cowsay -p ${env.COWSAY_PORT}:80 cowsay:${env.IMAGE_TAG} 80"
          sh 'sleep 5'
          def COWSAY_CONTAINER_IP = sh(script: 'docker inspect -f "{{ .NetworkSettings.Networks.bridge.Gateway }}" cowsay', returnStdout: true).trim()
          sh "curl http://${COWSAY_CONTAINER_IP}:80"
        }
      }
    }

    stage("Celan Up") {
      steps {
        script {
          sh 'docker rm -f cowsay'
        }
      }
    }
  }

  post {
    success {
      emailext body: "Your last build successed",
      subject: "SUCCESS",
      to: "${env.REGISTRY_USERNAME}"
    }

    failure {
      emailext body: "Your last build failed",
      subject: "FAILURE",
      to: "${env.REGISTRY_USERNAME}"
    }
  }
}
