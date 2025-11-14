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
      sh """
      docker compose build
      docker image tag "${DOCKER_ID}/${DOCKER_IMAGE}:movie.latest" "${DOCKER_ID}/${DOCKER_IMAGE}:movie.${DOCKER_TAG}"
      docker image tag "${DOCKER_ID}/${DOCKER_IMAGE}:cast.latest" "${DOCKER_ID}/${DOCKER_IMAGE}:cast.${DOCKER_TAG}"
      """
      }
    }
  }
  stage('Docker Run') {
    steps {
      script {
      sh """
      docker compose down
      docker compose up --no-build -d
      sleep 10
      """
      }
    }
  }
  stage('Test Acceptance') {
    steps {
      script {
      sh """
      curl localhost:8080/api/v1/movies/docs
      curl localhost:8080/api/v1/casts/docs
      """
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
        sh """
        docker login -u "$DOCKER_ID" -p "$DOCKER_PASS"
        # Push tagged images
        docker push "${DOCKER_ID}/${DOCKER_IMAGE}:movie.${DOCKER_TAG}"
        docker push "${DOCKER_ID}/${DOCKER_IMAGE}:cast.${DOCKER_TAG}"
        # Move latest tags
        docker push "${DOCKER_ID}/${DOCKER_IMAGE}:movie.latest"
        docker push "${DOCKER_ID}/${DOCKER_IMAGE}:cast.latest"
        """
      }
    }
  }
  // stage('Deploiement en dev'){
  //   environment {
  //   KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
  //   }
  //   steps {
  //     script {
  //     sh """
  //     rm -Rf .kube
  //     mkdir .kube
  //     ls
  //     cat $KUBECONFIG > .kube/config
  //     cp fastapi/values.yaml values.yml
  //     cat values.yml
  //     sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
  //     helm upgrade --install app fastapi --values=values.yml --namespace dev
  //     """
  //     }
  //   }
  // }
  // stage('Deploiement en staging'){
  //   environment {
  //   KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
  //   }
  //   steps {
  //     script {
  //     sh """
  //     rm -Rf .kube
  //     mkdir .kube
  //     ls
  //     cat $KUBECONFIG > .kube/config
  //     cp fastapi/values.yaml values.yml
  //     cat values.yml
  //     sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
  //     helm upgrade --install app fastapi --values=values.yml --namespace staging
  //     """
  //     }
  //   }
  // }
  // stage('Deploiement en prod'){
  //   environment {
  //     KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
  //     }
  //     steps {
  //       // Create an Approval Button with a timeout of 15minutes.
  //       // this require a manuel validation in order to deploy on production environment
  //       timeout(time: 15, unit: "MINUTES") {
  //         input message: 'Do you want to deploy in production ?', ok: 'Yes'
  //       }
  //       script {
  //         sh """
  //         rm -Rf .kube
  //         mkdir .kube
  //         ls
  //         cat $KUBECONFIG > .kube/config
  //         cp fastapi/values.yaml values.yml
  //         cat values.yml
  //         sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
  //         helm upgrade --install app fastapi --values=values.yml --namespace prod
  //         """
  //       }
  //     }
  //   }
  }
}