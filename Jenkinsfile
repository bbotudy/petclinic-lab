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
    stage('Docker Push') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-cred', usernameVariable: 'U', passwordVariable: 'P')]) {
          sh 'echo "$P" | docker login -u "$U" --password-stdin'
          sh 'docker push $IMAGE:$BUILD_NUMBER'
          sh 'docker push $IMAGE:latest'
        }
      }
    }
    stage('Deploy') {
      steps {
        withKubeConfig(credentialsId: 'k8s-token', serverUrl: 'https://kubernetes.default.svc', caCertificate: readFile('/var/run/secrets/kubernetes.io/serviceaccount/ca.crt')) {
          sh 'sed "s#$IMAGE:latest#$IMAGE:$BUILD_NUMBER#" deploy/petclinic.yml | kubectl apply -f -'
          sh 'kubectl apply -f deploy/petclinic-ingress.yml'
          sh 'kubectl rollout status deployment/petclinic -n was --timeout=300s'
        }
      }
    }
  }
}
