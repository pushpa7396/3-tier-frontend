pipeline {
    agent {
        label 'docker'
    }

    environment {
        KUBECONFIG_CREDENTIAL_ID = 'k8s-kubeconfig-dev'
        version = "frontend_${env.BUILD_NUMBER}"
        docker_image = "pushpa04/3-tier-app:${version}"
    }

    stages {
       stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/pushpa7396/3-tier-frontend.git'
            }
        }

       stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh "echo \"$DOCKER_PASSWORD\" | sudo docker login --username \"$DOCKER_USERNAME\" --password-stdin"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def dockerfilePath = '.'
                    sh "sudo docker build -t 'pushpa04/3-tier-app:${version}' ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    sh "sudo docker push 'pushpa04/3-tier-app:${version}'"
                }
            }
        } 
        stage('Security Scan') {
            steps {
                script {
                    def outputFilePath = "${env.WORKSPACE}/trivy_scan.txt"
                    def docker_image = "pushpa04/3-tier-app:${version}"
                    sh "sudo trivy image ${docker_image} > ${outputFilePath}"
                    sh "cat ${outputFilePath}"
                }
            }
        }
        stage('Cleanup Docker Images') {
            steps {
                script {
                    sh "sudo docker rmi -f ${env.docker_image}"
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {

                withCredentials([file(credentialsId: KUBECONFIG_CREDENTIAL_ID, variable: 'KUBECONFIG')]) {
                    script {
                      def kubeconfigPath = env.KUBECONFIG
                      withEnv(["VERSION=${env.version}"])
                      {
                         sh "echo ${VERSION}"
                         sh "export KUBECONFIG=${kubeconfigPath}"
                         //sh "kubectl scale deploy frontend --replicas=0 -n three-tier"
                         sh" sed -i 's/VERSION/${VERSION}/g' frontend.yml"
                         sh " cat frontend.yml"
                         sh "kubectl apply -f frontend.yml --validate=false"
                         sh "kubectl get pods -n three-tier"
                         sh "kubectl get svc -n three-tier"
                      }
                    }
                }
            }
        }
    }
}
