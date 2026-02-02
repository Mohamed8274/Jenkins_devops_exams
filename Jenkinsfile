pipeline {
environment { // Declaration of environment variables
DOCKER_ID = "united74" // replace this with your docker-id
DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
DOCKER_IMAGE_MOVIE = "movie-service"
DOCKER_IMAGE_CAST = "cast-service"
KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
}
agent any // Jenkins will be able to select all available agents
stages {
  stage('Docker Build'){ // docker build image stage
    parallel {
      stage('Docker Build movie-service'){
        steps {
          script {
          sh '''
          docker rm -f movie-service
          docker build -t $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG ./movie-service
          sleep 6
          '''
          }
        }
      }
      stage('Docker Build cast-service'){
        steps {
          script {
          sh '''
          docker rm -f cast-service
          docker build -t $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG ./cast-service
          sleep 6
          '''
          }
        }
      }
    }
  }
  stage('Docker run'){ // run container from our builded image
    parallel {
      stage('Docker run movie-service'){
        steps {
          script {
          sh '''
          docker run -d -p 8001:8000 --name movie-service $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG
          sleep 10
          '''
          }
        }
      }
      stage('Docker run cast-service'){
        steps {
          script {
          sh '''
          docker run -d -p 8002:8000 --name cast-service $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG
          sleep 10
          '''
          }
        }
      }
    }
  }
  stage('Test Acceptance'){ // we launch the curl command to validate that the container responds to the request
    parallel {
      stage('Test Acceptance movie_service'){
        steps {
          script {
          sh '''
          curl http://localhost:8001/api/v1/movies/
          '''
          // curl http://localhost:8000/api/v1/movies/ curl localhost
          }
        }
      }
      stage('Test Acceptance cast_service'){
        steps {
          script {
          sh '''
          curl http://localhost:8002/api/v1/cast/
          '''
          }
        }
      }
    }
  }
  stage('Docker Push'){ //we pass the built image to our docker hub account
    environment { 
      DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve  docker password from secret text called docker_hub_pass saved on jenkins
    }
    parallel {
      stage('Docker Push movie_service'){
        steps {
          script {
            sh '''
            docker login -u $DOCKER_ID -p $DOCKER_PASS
            docker push $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG
            '''
          }
        }
      }
      stage('Docker Push cast_service'){
        steps {
          script {
            sh '''
            docker login -u $DOCKER_ID -p $DOCKER_PASS
            docker push $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG
            '''
          }
        }
      }
    }
  }
  stage('Deploiement en dev'){
    steps {
      script {
      sh '''
      rm -Rf .kube
      mkdir .kube
      ls
      cat $KUBECONFIG > .kube/config
      cp fastapiapp/values.yaml values.yml
      cat values.yml
      sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
      helm upgrade --install fastapp fastapiapp --values=values.yml --namespace dev --set image.movie.repository="$DOCKER_ID/$DOCKER_IMAGE_MOVIE" --set image.cast.repository="$DOCKER_ID/$DOCKER_IMAGE_CAST" --set image.movie.tag="$DOCKER_TAG" --set image.cast.tag="$DOCKER_TAG"
      '''
      }
    }
  }
  stage('Deploiement en QA'){
    steps {
      script {
      sh '''
      rm -Rf .kube
      mkdir .kube
      ls
      cat $KUBECONFIG > .kube/config
      cp fastapiapp/values.yaml values.yml
      cat values.yml
      sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
      helm upgrade --install fastapp fastapiapp --values=values.yml --namespace qa --set image.movie.repository="$DOCKER_ID/$DOCKER_IMAGE_MOVIE" --set image.cast.repository="$DOCKER_ID/$DOCKER_IMAGE_CAST" --set image.movie.tag="$DOCKER_TAG" --set image.cast.tag="$DOCKER_TAG"
      '''
      }
    }
  }
  stage('Deploiement en staging'){
    steps {
      script {
      sh '''
      rm -Rf .kube
      mkdir .kube
      ls
      cat $KUBECONFIG > .kube/config
      cp fastapiapp/values.yaml values.yml
      cat values.yml
      sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
      helm upgrade --install fastapp fastapiapp --values=values.yml --namespace staging --set image.movie.repository="$DOCKER_ID/$DOCKER_IMAGE_MOVIE" --set image.cast.repository="$DOCKER_ID/$DOCKER_IMAGE_CAST" --set image.movie.tag="$DOCKER_TAG" --set image.cast.tag="$DOCKER_TAG"
      '''
      }
    }
  }
  stage('Deploiement en prod'){
    steps {
      // Create an Approval Button with a timeout of 15minutes.
      // this require a manuel validation in order to deploy on production environment
      timeout(time: 15, unit: "MINUTES") {
        input message: 'Do you want to deploy in production ?', ok: 'Yes'
      }
      script {
        sh '''
        rm -Rf .kube
        mkdir .kube
        ls
        cat $KUBECONFIG > .kube/config
        cp fastapiapp/values.yaml values.yml
        cat values.yml
        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
        helm upgrade --install fastapp fastapiapp --values=values.yml --namespace prod --set image.movie.repository="$DOCKER_ID/$DOCKER_IMAGE_MOVIE" --set image.cast.repository="$DOCKER_ID/$DOCKER_IMAGE_CAST" --set image.movie.tag="$DOCKER_TAG" --set image.cast.tag="$DOCKER_TAG"
        '''
      }
    }
  }
}
}