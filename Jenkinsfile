pipeline {
environment {
  DOCKER_ID = "kasyc"
  DOCKER_IMAGE = "jenkins-devops-exam"
  DOCKER_TAG = "v.${BUILD_ID}.0"
}
agent any
stages {
  stage('Docker Build') {
    steps {
      script {
      sh '''
      docker compose build
      docker image tag "${DOCKER_ID}/${DOCKER_IMAGE}:movie.latest" "${DOCKER_ID}/${DOCKER_IMAGE}:movie.${DOCKER_TAG}"
      docker image tag "${DOCKER_ID}/${DOCKER_IMAGE}:cast.latest" "${DOCKER_ID}/${DOCKER_IMAGE}:cast.${DOCKER_TAG}"
      '''
      }
    }
  }
  stage('Docker Run') {
    steps {
      script {
      sh '''
      docker compose down
      docker compose up --no-build -d
      sleep 10
      '''
      }
    }
  }
  stage('Test Acceptance') {
    steps {
      script {
      sh '''
      curl localhost:8080/api/v1/movies/docs
      curl localhost:8080/api/v1/casts/docs
      '''
      }
    }
  }
  stage('Docker Push') {
    environment
    {
      DOCKER_PASS = credentials("DOCKER_HUB_PASS")
    }
    steps {
      script {
      sh '''
      docker login -u "$DOCKER_ID" -p "$DOCKER_PASS"
      # Push tagged images
      docker push "${DOCKER_ID}/${DOCKER_IMAGE}:movie.${DOCKER_TAG}"
      docker push "${DOCKER_ID}/${DOCKER_IMAGE}:cast.${DOCKER_TAG}"
      # Move latest tags
      docker push "${DOCKER_ID}/${DOCKER_IMAGE}:movie.latest"
      docker push "${DOCKER_ID}/${DOCKER_IMAGE}:cast.latest"
      '''
      }
    }
  }
  stage('Configure k8s') {
    environment {
      KUBECONFIG = credentials("config")
    }
    steps {
      script {
      sh '''
      rm -Rf .kube
      mkdir .kube
      cat $KUBECONFIG > .kube/config
      '''
      }
    }
  }
  stage('Deploy dev') {
    environment {
      KUBECONFIG = credentials("config")
      NAMESPACE = "dev"
      NODEPORT = "30001"
    }
    steps {
      script {
      sh '''
      helm upgrade --install app charts/ --values=charts/values.yaml \
        --namespace "$NAMESPACE" \
        --set image.movieTag="movie.${DOCKER_TAG}" \
        --set image.castTag="cast.${DOCKER_TAG}" \
        --set service.nodePort="$NODEPORT"
      '''
      }
    }
  }
  stage('Deploy qa') {
    environment {
      KUBECONFIG = credentials("config")
      NAMESPACE = "qa"
      NODEPORT = "30002"
    }
    steps {
      script {
      sh '''
      helm upgrade --install app charts/ --values=charts/values.yaml \
        --namespace "$NAMESPACE" \
        --set image.movieTag="movie.${DOCKER_TAG}" \
        --set image.castTag="cast.${DOCKER_TAG}" \
        --set service.nodePort="$NODEPORT"
      '''
      }
    }
  }
  stage('Deploy staging') {
    environment {
      KUBECONFIG = credentials("config")
      NAMESPACE = "staging"
      NODEPORT = "30003"
    }
    steps {
      script {
      sh '''
      helm upgrade --install app charts/ --values=charts/values.yaml \
        --namespace "$NAMESPACE" \
        --set image.movieTag="movie.${DOCKER_TAG}" \
        --set image.castTag="cast.${DOCKER_TAG}" \
        --set service.nodePort="$NODEPORT"
      '''
      }
    }
  }
  stage('Deploy prod') {
    when {
      // Deploy to production only from the master branch.
      environment name: 'GIT_BRANCH', value: 'origin/master'
    }
    environment {
      KUBECONFIG = credentials("config")
      NAMESPACE = "prod"
      NODEPORT = "30000"
    }
    steps {
      // Create a manual approval button with a 15 minutes timeout.
      timeout(time: 15, unit: "MINUTES") {
        input message: 'Do you want to deploy in production ?', ok: 'Yes'
      }
      script {
      sh '''
      helm upgrade --install app charts/ --values=charts/values.yaml \
        --namespace "$NAMESPACE" \
        --set image.movieTag="movie.${DOCKER_TAG}" \
        --set image.castTag="cast.${DOCKER_TAG}" \
        --set service.nodePort="$NODEPORT"
      '''
      }
    }
  }
}
}