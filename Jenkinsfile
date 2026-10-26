pipeline {
  agent any
  environment {
    IMAGE = 'ldy88/petclinic'
    DOCKERHUB_CRED = credentials('dockerhub-cred')
  }
  stages {
    stage('Maven Build') {
      steps {
        sh 'chmod +x mvnw && ./mvnw -B clean package -DskipTests'
      }
    }
    stage('Docker Build') {
      steps {
        sh 'docker build -t $IMAGE:$BUILD_NUMBER -t $IMAGE:latest .'
      }
    }
    stage('Docker Push') {
      steps {
        sh 'echo $DOCKERHUB_CRED_PSW | docker login -u $DOCKERHUB_CRED_USR --password-stdin'
        sh 'docker push $IMAGE:$BUILD_NUMBER'
        sh 'docker push $IMAGE:latest'
      }
    }
  }
  post {
    always {
      sh 'docker logout || true'
    }
  }
}
