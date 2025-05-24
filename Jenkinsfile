pipeline {
  agent any

  environment {
    DOCKER_HUB_USER = 'vantaicn'
    IMAGE_TAG = ''
    HELM_REPO_DIR = 'spring-petclinic-microservices-gitops'
    TARGET_SERVICES = ''
  }

  stages {

    stage('Prepare') {
      steps {
        script {
          if (env.TAG_NAME) {
            IMAGE_TAG = env.TAG_NAME
          } else if (env.GIT_COMMIT) {
            IMAGE_TAG = env.GIT_COMMIT.take(7)
          }
          echo "Image tag to be used: ${IMAGE_TAG}"
        }
      }
    }

    stage('Detect Changed Service') {
      steps {
        script {
          def allServices = [
            'spring-petclinic-admin-server',
            'spring-petclinic-customers-service',
            'spring-petclinic-vets-service',
            'spring-petclinic-visits-service',
            'spring-petclinic-config-server',
            'spring-petclinic-discovery-server',
            'spring-petclinic-api-gateway'
          ]

          if (env.TAG_NAME) {
            echo "Tag detected (${env.TAG_NAME}) → building all services."
            TARGET_SERVICES = allServices.join(',')
          } else {
            def changedFiles = sh(script: "git diff --name-only HEAD~1", returnStdout: true).trim()
            def services = [] as Set
            changedFiles.split('\n').each { file ->
              def folder = file.split('/')[0]
              if (folder.startsWith("spring-petclinic-") && allServices.contains(folder)) {
                services << folder
              }
            }

            if (services.isEmpty()) {
              error "❌ No changed services detected. Stopping pipeline."
            }

            TARGET_SERVICES = services.join(',')
            echo "Changed services: ${TARGET_SERVICES}"
          }
        }
      }
    }

    stage('Build Docker Images') {
      steps {
        script {
          TARGET_SERVICES.split(',').each { svc ->
            dir("${svc}") {
              sh "../mvnw clean install -P buildDocker -Ddocker.image.prefix=${DOCKER_HUB_USER}"
            }
          }
        }
      }
    }

    stage('Push Docker Images') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-vantaicn', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          script {
            sh 'echo "${DOCKER_PASS}" | docker login -u "${DOCKER_USER}" --password-stdin'
            TARGET_SERVICES.split(',').each { svc ->
              sh """
                docker tag ${DOCKER_HUB_USER}/${svc}:latest ${DOCKER_HUB_USER}/${svc}:${IMAGE_TAG}
                docker push ${DOCKER_HUB_USER}/${svc}:${IMAGE_TAG}
              """
            }
          }
        }
      }
    }

    stage('Checkout Helm Repo') {
      when {
        anyOf {
          branch 'main'
          expression { return env.TAG_NAME?.startsWith('v') }
        }
      }
      steps {
        dir("${HELM_REPO_DIR}") {
          git url: 'https://github.com/vantaicn/spring-petclinic-microservices-gitops.git', credentialsId: 'github-vantaicn', branch: 'main'
        }
      }
    }

    stage('Update Helm values') {
      when {
        anyOf {
          branch 'main'
          expression { return env.TAG_NAME?.startsWith('v') }
        }
      }
      steps {
        script {
          def envFile = env.TAG_NAME?.startsWith('v') ? 'values-staging.yaml' : 'values-dev.yaml'
          def helmValuesFile = "${HELM_REPO_DIR}/${envFile}"

          echo "Updating image tag in ${helmValuesFile}"
          TARGET_SERVICES.split(',').each { svc ->
            sh """
              yq e '.services["${svc.replace("spring-petclinic-", "")}"].tag = "${IMAGE_TAG}"' -i ${helmValuesFile}
            """
          }
          env.HELM_VALUES_UPDATED = "true"
        }
      }
    }

    stage('Commit & Push to Git') {
      when {
        expression { return env.HELM_VALUES_UPDATED == "true" }
      }
      steps {
        dir("${HELM_REPO_DIR}") {
          withCredentials([usernamePassword(credentialsId: 'github-vantaicn', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')]) {
            sh """
              git config user.email "ci@example.com"
              git config user.name "jenkins"
              git add .
              git commit -m "Update image tag(s): ${IMAGE_TAG}" || echo "No changes to commit"
              git push https://${GIT_USER}:${GIT_PASS}@github.com/vantaicn/spring-petclinic-microservices-gitops.git HEAD:main
            """
          }
        }
      }
    }
  }
}
