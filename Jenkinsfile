pipeline {
  agent any
  environment {
    DH_USER = 'ldy88'
    IMAGE = "${DH_USER}/petclinic"
  }
  stages {
    stage('Build') {
      steps { sh 'chmod +x mvnw && ./mvnw -B -DskipTests clean package' }
    }
    stage('Docker Build') {
      steps { sh 'docker build -t $IMAGE:$BUILD_NUMBER -t $IMAGE:latest .' }
    }
    stage('Docker Login') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-cred', usernameVariable: 'U', passwordVariable: 'P')]) {
          sh 'echo "$P" | docker login -u "$U" --password-stdin'
        }
      }
    }
    stage('Docker Push') {
      steps {
        sh 'docker push $IMAGE:$BUILD_NUMBER'
        sh 'docker push $IMAGE:latest'
      }
    }
    stage('Deploy') {
      steps {
        sh 'sed "s#$IMAGE:latest#$IMAGE:$BUILD_NUMBER#" deploy/deployment.yml | kubectl apply -f -'
        sh 'kubectl apply -f deploy/service.yml -f deploy/ingress.yml'
        sh 'kubectl rollout status deployment/petclinic -n was --timeout=300s'
      }
    }
  }
}
