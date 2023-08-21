pipeline {
    agent any
    environment {
        //be sure to replace "bhavukm" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "kunnvij/devops_training"
        DOCKERHUB_CREDENTIALS = credentials('devops_training')
    }
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    //app = docker.build(DOCKER_IMAGE_NAME)
                    //app.inside {
                    //    sh 'echo Hello, World!'
                    //}
                    sh 'docker build -t kunnvij/devops_training .'
                }
            }
        }
        stage('Login') {
          steps {
            sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
          }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    //docker.withRegistry('https://hub.docker.com/kunnvij/devops_training', 'devops_training') {
                     //   app.push("${env.BUILD_NUMBER}")
                       // app.push("latest")
                    //}
                    sh 'docker push kunnvij/devops_training'
                }
            }
        }
        stage('CanaryDeploy') {
            when {
                branch 'master'
            }
            environment { 
                CANARY_REPLICAS = 1
            }
            steps {
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube-canary.yml',
                    enableConfigSubstitution: true
                )
            }
        }
        stage('DeployToProduction') {
            environment { 
                CANARY_REPLICAS = 0
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube-canary.yml',
                    enableConfigSubstitution: true
                )
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube.yml',
                    enableConfigSubstitution: true
                )
            }
        }
    }
}
