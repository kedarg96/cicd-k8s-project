pipeline {
  agent any

  environment {
    IMAGE = "kedarg96/myapp:${BUILD_NUMBER}"
  }

  stages {
    stage('Build & Test') {
      steps {
        dir('app') { sh 'mvn -B clean package' }
      }
    }

    stage('Docker Build') {
      steps {
        dir('app') { sh 'docker build -t $IMAGE .' }
      }
    }

    stage('Push Image') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub',
                         usernameVariable: 'DH_USER', passwordVariable: 'DH_PASS')]) {
          sh '''
            echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin
            docker push $IMAGE
          '''
        }
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        sh '''
          sed "s|IMAGE_PLACEHOLDER|$IMAGE|" k8s/deployment.yaml | kubectl apply -f -
          kubectl rollout status deployment/myapp --timeout=180s
        '''
      }
    }
  }

  post {
    always { sh 'docker logout || true' }
  }
}
