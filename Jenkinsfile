pipeline {
    agent any

    environment {
        IMAGE_NAME = "manojkrishnappa/fullstack:${GIT_COMMIT}"
        AWS_REGION = "ap-south-2"
        CLUSTER_NAME = "my-complete-eks"
        NAMESPACE = "microdegree"
    }

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

        stage('sonarqube-stage'){
            steps{
                sh"""
                mvn sonar:sonar \
                -Dsonar.projectKey=devops \
                -Dsonar.host.url=http://3.90.207.159:9000 \
                -Dsonar.login=3bc7e2fd3433144539118dc582575ad22bcd2d0d
                """
            }
        }
        stage('Build & Tag Docker Image') {
            steps {
                script {
                    sh 'printenv'
                    sh "docker build -t manojkrishnappa/fullstack:${GIT_COMMIT} ."
                }
            }
        }

        stage('Docker Image Scan') {
            steps {
                script {
                    sh "trivy image --format table -o trivy-image-report.html manojkrishnappa/fullstack:${GIT_COMMIT}"
                }
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
                    sh "docker push manojkrishnappa/fullstack:${GIT_COMMIT}"
                }
            }
        }
        
        stage('Updating the Cluster') {
            steps {
                script {
                    sh "aws eks update-kubeconfig --region ${AWS_REGION} --name ${CLUSTER_NAME}"
                }
            }
        }
        
        stage('Deploy To Kubernetes') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'my-complete-eks', contextName: '', credentialsId: 'kube', namespace: 'sadiq', restrictKubeConfigAccess: false, serverUrl: 'https://2E2EDEEBFD5FB5EFAF01F598B7753431.yl4.ap-south-2.eks.amazonaws.com') {
                    sh "sed -i 's|replace|${IMAGE_NAME}|g' deployment.yml"
                    sh "kubectl apply -f deployment.yml -n ${NAMESPACE}"
                }
            }
        }

        stage('Verify the Deployment') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'my-complete-eks', contextName: '', credentialsId: 'kube', namespace: 'sadiq', restrictKubeConfigAccess: false, serverUrl: 'https:https://2E2EDEEBFD5FB5EFAF01F598B7753431.yl4.ap-south-2.eks.amazonaws.com') {
                    sh "kubectl get pods -n microdegree"
                    sh "kubectl get svc -n microdegree"
                }
            }
        }
    }

    post {
        always {
            script {
                def jobName = env.JOB_NAME
                def buildNumber = env.BUILD_NUMBER
                def pipelineStatus = currentBuild.result ?: 'UNKNOWN'
                def bannerColor = pipelineStatus.toUpperCase() == 'SUCCESS' ? 'green' : 'red'

                def body = """
                    <html>
                    <body>
                    <div style="border: 4px solid ${bannerColor}; padding: 10px;">
                    <h2>${jobName} - Build ${buildNumber}</h2>
                    <div style="background-color: ${bannerColor}; padding: 10px;">
                    <h3 style="color: white;">Pipeline Status: ${pipelineStatus.toUpperCase()}</h3>
                    </div>
                    <p>Check the <a href="${BUILD_URL}">console output</a>.</p>
                    </div>
                    </body>
                    </html>
                """

                emailext (
                    subject: "${jobName} - Build ${buildNumber} - ${pipelineStatus.toUpperCase()}",
                    body: body,
                    to: 'mohamedsadiq9741@gmail.com',
                    from: 'mohamedsadiq9741@gmail.com',
                    replyTo: 'mohamedsadiq9741@gmail.com',
                    mimeType: 'text/html',
                    attachmentsPattern: 'trivy-image-report.html'
                )
            }
        }
    }
}
