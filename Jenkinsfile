pipeline {
  agent any

  environment {
    DOCKERHUB_CRED = 'dockerhub-cred-id'
    JFROG_CRED = 'jfrog-cred-id'
    SONAR_TOKEN = credentials('sonar-token-id')
    IMAGE_NAME = "yourdockerhubusername/springboot-hello-world"
  }

  options {
    buildDiscarder(logRotator(numToKeepStr: '20'))
    timestamps()
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build (Maven)') {
      steps {
        sh 'mvn -B -DskipTests clean package'
      }
      post {
        success {
          archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
        }
      }
    }

    stage('Run Unit Tests') {
      steps {
        sh 'mvn test'
        junit 'target/surefire-reports/*.xml'
      }
    }

    stage('SonarQube Analysis') {
      steps {
        withSonarQubeEnv('SonarQube') {
          sh "mvn sonar:sonar -Dsonar.login=${SONAR_TOKEN}"
        }
      }
    }

    stage('Upload to JFrog') {
      steps {
        withCredentials([usernamePassword(credentialsId: "${JFROG_CRED}", usernameVariable: 'JF_USER', passwordVariable: 'JF_PASS')]) {
          sh '''
            ARTIFACT=target/*.jar
            curl -u $JF_USER:$JF_PASS -T ${ARTIFACT} "https://your-jfrog-url/artifactory/libs-release-local/springboot/${BUILD_NUMBER}/springboot-hello-world-${BUILD_NUMBER}.jar"
          '''
        }
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          def tag = "${env.BUILD_NUMBER}"
          sh "docker build -t ${IMAGE_NAME}:${tag} ."
          sh "docker tag ${IMAGE_NAME}:${tag} ${IMAGE_NAME}:latest"
          env.IMAGE_TAG = tag
        }
      }
    }

    stage('Trivy Security Scan') {
      steps {
        sh "trivy image --exit-code 0 --severity HIGH,CRITICAL ${IMAGE_NAME}:${IMAGE_TAG} || true"
      }
    }

    stage('Push to DockerHub') {
      steps {
        withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CRED}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          sh '''
            echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
            docker push ${IMAGE_NAME}:${IMAGE_TAG}
            docker push ${IMAGE_NAME}:latest
          '''
        }
      }
    }

    stage('Cleanup') {
      steps {
        sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG} || true"
        sh "docker rmi ${IMAGE_NAME}:latest || true"
      }
    }

    stage('Update K8s Manifest for ArgoCD') {
      steps {
        sh '''
          sed -i "s|image: .*|image: ${IMAGE_NAME}:${IMAGE_TAG}|g" k8s/deployment.yaml
          git config user.email "jenkins@example.com"
          git config user.name "jenkins"
          git add k8s/deployment.yaml
          git commit -m "Update image tag to ${IMAGE_TAG}" || true
          git push origin main || true
        '''
      }
    }
  }

  post {
    always {
      cleanWs()
    }
    success {
      echo "✅ Build ${env.BUILD_NUMBER} completed successfully!"
    }
    failure {
      echo "❌ Build failed. Check Jenkins logs."
    }
  }
}
