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
  

      stages {
      stage('Verify Branch') {
        steps {
          sshagent(['ssh_to_gitlab']) {
            script {
              // Parameterized Build
              if (params.MAJOR_VERSION && params.MINOR_VERSION) {
                def branchName = "release/${params.MAJOR_VERSION}.${params.MINOR_VERSION}"
                def branchExists = sh ( script: "git ls-remote --heads origin \"$branchName\" | wc -l", returnStdout: true).trim().toInteger() > 0
                if (!branchExists) {
                  echo "The branch ${branchName} does not exist. Creating it..."
                  sh 'git checkout master'
                  sh "git checkout -b ${branchName} || git checkout ${branchName}"
                  writeFile file: "${env.VERSION_FILE_NAME}", text: "${params.MAJOR_VERSION}.${params.MINOR_VERSION}"
                  sh "git add ${env.VERSION_FILE_NAME}"
                  sh "git config --global user.email \"${env.REGISTRY_USERNAME}\""
                  sh "git config --global user.name \"${env.REGISTRY_NAME}\""
                  sh "git commit -m 'Initialized ${branchName}'"
                  sh "git push origin ${branchName}"
                }
                BRANCH_NAME = branchName
              
              // Push Triggered Build
              } else {
                def branchName = env.GIT_BRANCH
                if (branchName.startsWith('origin/release/') || branchName.startsWith('release/')) {
                  def version = branchName.split('/')[-1]
                  MAJOR_VERSION = version.split('\\.')[0]
                  MINOR_VERSION = version.split('\\.')[1]
                }
                BRANCH_NAME  = branchName
              }
            }
          }
        }
      }

      stage('Checkout') {
        steps {
          script {
            git branch: "${BRANCH_NAME}", credentialsId: "${env.SSH_CREDENTIALS_ID}", url: "${env.GIT_URL}"
            FOR_RELEASE = sh (script: "cat ${env.VERSION_FILE_NAME} | grep -i \"not for release\" | wc -l", returnStdout: true).trim().toInteger() == 0
          }
        }
      }

      stage('Calculate and Set Patch Version') {
        steps {
          sshagent(['ssh_to_gitlab']) {
            script {
              if (FOR_RELEASE) {
                sh 'git fetch --tags'
                def latestPatch = sh( script: "git tag -l ${MAJOR_VERSION}.${MINOR_VERSION}.* | sort -V | tail -n1 | cut -d '.' -f 3", returnStdout: true).trim()
                if (latestPatch.length() != 0) {
                  PATCH_NUMBER = latestPatch.toInteger() + 1
                }
                writeFile file: "${VERSION_FILE_NAME}", text: "${MAJOR_VERSION}.${MINOR_VERSION}.${PATCH_NUMBER}"
              }
            }
          }
        }
      }

      stage('Build') {
        steps {
          script {
            def tag = FOR_RELEASE ? "${MAJOR_VERSION}.${MINOR_VERSION}.${PATCH_NUMBER}" : "not_for_release"
            sh "docker build -t cowsay:${tag} ."
          }
        }
      }
      
      stage('Test') {
        steps {
          script {
            def tag = FOR_RELEASE ? "${MAJOR_VERSION}.${MINOR_VERSION}.${PATCH_NUMBER}" : "not_for_release"
            sh 'docker rm -f cowsay'
            sh "docker run --rm -d --name cowsay -p ${env.COWSAY_PORT}:80 cowsay:${tag} 80"
            sh 'sleep 5'
            def COWSAY_CONTAINER_IP = sh(script: 'docker inspect -f "{{ .NetworkSettings.Networks.bridge.Gateway }}" cowsay', returnStdout: true).trim()
            sh "curl http://${COWSAY_CONTAINER_IP}:80"
            sh 'docker rm -f cowsay'
          }
        }
      }

      stage('Publish') {
        steps {
          script {
            if (FOR_RELEASE) {
              withCredentials(
                [string(credentialsId: "${env.PERSONAL_ACCESS_TOKEN_ID}", variable: 'TOKEN')]
              ) {
                def tag = "${MAJOR_VERSION}.${MINOR_VERSION}.${PATCH_NUMBER}"
                sh """
                  echo "${TOKEN}" | docker login "${env.REGISTRY_URL}" -u "${env.REGISTRY_USERNAME}" --password-stdin
                  docker build -t "${env.REGISTRY_PROJECT_URL}:${tag}" .
                  docker push "${env.REGISTRY_PROJECT_URL}:${tag}"
                """
              }
            }
          }
        }
      }

      stage('Clean Up') {
        steps {
          script {
            sshagent(['ssh_to_gitlab']) {
              script {
                if (FOR_RELEASE) {
                  def tag = "${MAJOR_VERSION}.${MINOR_VERSION}.${PATCH_NUMBER}"
                  sh "git restore ${env.VERSION_FILE_NAME}"
                  sh "git tag ${tag}"
                  sh "git push origin tag ${tag}"
                }
              }
            }
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
