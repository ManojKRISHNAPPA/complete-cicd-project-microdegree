pipeline {
    agent any

    tools {
        jdk 'java-17'
        maven 'maven'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/ManojKRISHNAPPA/complete-cicd-project-microdegree.git'
            }
        }

        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }

        stage('Build') {
            steps {
                sh "mvn package"
            }
        }

        stage('Build & Tag Docker Image') {
            steps {
                script {
                    sh "docker build -t manojkrishnappa/fullstack:20241003 ."
                }
            }
        }

        stage('Docker Image Scan') {
            steps {
                sh "trivy image --format table -o trivy-image-report.html manojkrishnappa/fullstack:20241003"
            }
        }

        stage('Login to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh "echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin"
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    sh "docker push manojkrishnappa/fullstack:20241003"
                }
            }
        }

        stage('Deploy To Kubernetes') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: ' microdegree-cluster', contextName: '', credentialsId: 'kube', namespace: ' microdegree', restrictKubeConfigAccess: false, serverUrl: 'https://4AB4BE52402C9FB1670ECBA301DDE4C2.gr7.us-east-1.eks.amazonaws.com') {
                    sh "aws eks update-kubeconfig --region us-east-1 --name microdegree-cluster"
                    sh "kubectl get pods -n microdegree"
                    sh "kubectl apply -f deployment.yml -n microdegree"
                }
            }
        }

        stage('Verify the Deployment') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: ' microdegree-cluster', contextName: '', credentialsId: 'kube', namespace: ' microdegree', restrictKubeConfigAccess: false, serverUrl: 'https://4AB4BE52402C9FB1670ECBA301DDE4C2.gr7.us-east-1.eks.amazonaws.com') {
                    sh "kubectl get pods -n microdegree"
                    sh "kubectl get svc -n microdegree"
                }
            }
        }
    }
}

