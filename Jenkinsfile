pipeline{

  agent {
      kubernetes {
        yaml '''
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spring-boot-app
  labels:
    app: spring-boot-app
    type: back-end
spec:
  template:
    metadata:
      name: spring-boot-app
      labels:
        app: spring-boot
        type: back-end
    spec:
      containers:
        - name: spring-boot-app
          image: lhamaoka/jenkins-nodo-nodejs-bootcamp:1.0
          imagePullPolicy: Always
  replicas: 1
  selector:
    matchLabels:
      app: spring-boot
      type: back-end
        '''
        defaultContainer 'shell'
      }
  }

  environment {
    registryCredential='docker-hub-credentials'
    registryFrontend = 'lhamaoka/angular-14-app'
  }

  stages {
    stage('Build') {
      steps {
        sh 'npm install'
        sh 'npm run build &'
        sleep 15
      }
    }

    stage('Push Image to Docker Hub') {
      steps {
        script {
          dockerImage = docker.build registryFrontend + ":$BUILD_NUMBER"
          docker.withRegistry( '', registryCredential) {
            dockerImage.push()
          }
        }
      }
    }

    stage('Push Image latest to Docker Hub') {
      steps {
        script {
          dockerImage = docker.build registryFrontend + ":latest"
          docker.withRegistry( '', registryCredential) {
            dockerImage.push()
          }
        }
      }
    }

    stage('Deploy to K8s') {

      steps{
        script {
          if(fileExists("configuracion")){
            sh 'rm -r configuracion'
          }
        }
        sh 'git clone https://github.com/lhamaoka/kubernetes-helm-docker-config.git configuracion --branch test-implementation'
        sh 'kubectl apply -f configuracion/kubernetes-deployment/angular-14-app/manifest.yml -n default --kubeconfig=configuracion/kubernetes-config/config'
      }

    }
  }

  post {
    always {
      sh 'docker logout'
    }
  }
}
