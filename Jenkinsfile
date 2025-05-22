pipeline {
  agent any

  environment {
    DOCKER_HUB_USER = 'vantaicn'
    IMAGE_TAG = ''
  }

  stages {
    stage('Prepare') {
      steps {
        script {
          if (env.GIT_COMMIT) {
            IMAGE_TAG = env.GIT_COMMIT.take(7)
          }

          if (env.GIT_TAG_NAME) {
            IMAGE_TAG = env.GIT_TAG_NAME
          }

          echo "Image tag: ${IMAGE_TAG}"
        }
      }
    }

    stage('Detect Changed Service') {
      steps {
        script {
          def changedFiles = sh(script: "git diff --name-only HEAD~1", returnStdout: true).trim()
          def matchedService = ''

          changedFiles.split('\n').each { file ->
            if (file.contains("spring-petclinic-") && file.contains("-service/")) {
              matchedService = file.split('/')[0]
            }
          }

          if (!matchedService) {
            error("Không phát hiện service nào bị thay đổi.")
          }

          env.TARGET_SERVICE = matchedService
          echo "Changed services: ${env.TARGET_SERVICE}"
        }
      }
    }

    // stage('Build Docker Image') {
    //   steps {
    //     dir("${env.TARGET_SERVICE}") {
    //       sh """
    //         ../mvnw clean install -P buildDocker -Dspring.profiles.active=native -Ddocker.image.prefix=${DOCKER_HUB_USER} -Ddocker.image.tag=${IMAGE_TAG}
    //         docker images | grep spring-petclinic
    //       """
    //     }
    //   }
    // }

    // stage('Push Docker Image') {
    //     steps {
    //       withCredentials([usernamePassword(credentialsId: 'docker_hub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
    //         script {
    //             sh """
    //             echo "${DOCKER_PASS}" | docker login -u "${DOCKER_USER}" --password-stdin
    //             docker tag ${DOCKER_HUB_USER}/${env.TARGET_SERVICE}:latest ${DOCKER_HUB_USER}/${env.TARGET_SERVICE}:${IMAGE_TAG}
    //             docker push ${DOCKER_USER}/${env.TARGET_SERVICE}:${IMAGE_TAG}
    //             """
    //         }
    //       }
    //     }
    // }

    // stage('Checkout Helm Repo') {
    //   steps {
    //     dir('helm-lab2') {
    //       git url: 'https://github.com/vantaicn/helm-lab2.git', credentialsId: 'github-token', branch: 'main'
    //     }
    //     script {
    //       env.HELM_REPO_DIR = 'helm-lab2'
    //     }
    //   }
    // }

    // stage('Update Helm values') {
    //   steps {
    //     script {
    //       def envFile = ''
    //       if (env.GIT_BRANCH == 'main') {
    //         envFile = 'values-dev.yaml'
    //       } else if (env.GIT_TAG_NAME?.startsWith('v')) {
    //         envFile = 'values-staging.yaml'
    //       } else {
    //         error("Không xác định môi trường (dev/staging)")
    //       }

    //       def helmValuesFile = "${HELM_REPO_DIR}/${envFile}"
    //       echo "Cập nhật tag cho service ${env.TARGET_SERVICE} trong file ${helmValuesFile}"

    //       sh """
    //         yq e '.services["${env.TARGET_SERVICE}"].tag = "${IMAGE_TAG}"' -i ${helmValuesFile}
    //       """
    //     }
    //   }
    // }

    // stage('Commit & Push to Git') {
    //   steps {
    //     dir("${HELM_REPO_DIR}") {
    //       withCredentials([usernamePassword(credentialsId: 'github-token', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')]) {
    //         sh """
    //           git config user.email "ci@example.com"
    //           git config user.name "jenkins"
    //           git add .
    //           git commit -m "Update image for ${env.TARGET_SERVICE} to ${IMAGE_TAG}" || echo "Nothing to commit"
    //           git push https://${GIT_USER}:${GIT_PASS}@github.com/vantaicn/helm-lab2.git HEAD:main
    //         """
    //       }
    //     }
    //   }
    // }
  }
}