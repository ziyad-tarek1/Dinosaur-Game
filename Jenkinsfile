## Success

pipeline {
    agent any
    tools {
        jdk 'jdk17'
    }

    environment {
        GITHUB_CREDENTIALS = 'Git-cred' 
        GIT_REPO_NAME = "Dinosaur-Game"
        GIT_REPO_URL  = "https://github.com/ziyad-tarek1/Dinosaur-Game.git"
        DOCKER_CREDENTIALS_ID = 'DockerHub-Cred'
        DOCKERHUB_REPO = 'ziyadtarek99/myreact-app'
        K8S_CRED_ID = 'myminikube-cred'
    }

    stages {
        /*  stage('Clean Workspace') {
                steps {
                    cleanWs()
                }
            } */

        stage('Checkout') {
            steps {
                git branch: 'main', credentialsId: "${GITHUB_CREDENTIALS}", url: 'https://github.com/ziyad-tarek1/myreact-app.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                dir('App') { 
                    sh "npm install"
                }
            }
        }

        stage('Build') {
            steps {
                dir('App') { 
                    sh "npm run build"
                }
            }
        }

        stage('Trivy Scan FS') {
            steps {
                script {
                    def scanResultsFile = "trivy-scan-results.json"
                    dir('App') {
                        // Run Trivy as a container to scan the FS 
                        sh """
                         docker run --rm \
                            -v \$(pwd):/app \
                            aquasec/trivy:latest fs \
                            --severity HIGH,CRITICAL \
                            --format json \
                            --output /app/${scanResultsFile} \
                            /app
                        """
                    }
                    archiveArtifacts artifacts: "App/${scanResultsFile}", fingerprint: true
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                dir('App') { // Change to the directory containing the Dockerfile
                    script {
                        // Calculate the new tag (increment by 1.0 for each build)
                        def newTag = "${env.BUILD_NUMBER}.0"
                        env.IMAGE_TAG = newTag
                        
                        // Build the Docker image with the new tag
                        docker.build("${DOCKERHUB_REPO}:${newTag}")
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    // Push the Docker image to Docker Hub with the new tag
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDENTIALS_ID) {
                        docker.image("${DOCKERHUB_REPO}:${IMAGE_TAG}").push()
                        
                        // Tag the image as latest and push it
                        docker.image("${DOCKERHUB_REPO}:${IMAGE_TAG}").push('latest')
                    }
                }
            }
        }

        stage('Trivy Scan Image') {
            steps {
                script {
                    def scanResultsImage = "trivy-scan-image-results.json"
                    dir('App') {
                        // Run Trivy as a container to scan the container image
                        sh """
                            docker run --rm \
                                -v \$(pwd):/data \
                                aquasec/trivy:latest image \
                                --severity HIGH,CRITICAL \
                                --format json \
                                --output /data/${scanResultsImage} \
                                ${DOCKERHUB_REPO}:${IMAGE_TAG}
                        """
                    }
                    archiveArtifacts artifacts: "App/${scanResultsImage}", fingerprint: true
                }
            }
        }

        // this is the new stage kindly debug it
        stage('github repo update') {
            steps {
                script {

                    //update the deployment file an push it to git 
                    
                        sh 'sed -i "s|image:.*|image: ${DOCKERHUB_REPO}:${IMAGE_TAG}|" k8s/deployment.yaml'
                        sh 'git add k8s/deployment.yaml'
                        sh "git commit -m 'Update deployment image to ziyadtarek99/myreact-app:latest'"
                        // edit the below line to match my jenkins configration
                        sh "git push https://${GIT_USERNAME}:${GITHUB_TOKEN}@${GIT_REPO_URL} HEAD:main"

                }
            }
        }

        // this stage works fine
        stage('Deploy to Minikube') {
            steps {
                withKubeConfig(credentialsId: "${env.K8S_CRED_ID}") {
                    // Apply the ArgoCD application.yaml to start the CD process
                    sh 'kubectl apply -f application.yaml'
                }
            }
        }
  

    }

    post {
        always {
            emailext(
                subject: "${currentBuild.currentResult}: Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\nMore info at: ${env.BUILD_URL}",
                body: "Please find the attached file created by Jenkins.",
                to: 'ziyadtarek180@gmail.com',
                from: 'ziyadtarek180@gmail.com',
                replyTo: 'ziyadtarek180@gmail.com',
                attachLog: true,
                attachmentsPattern: "App/trivy-scan-results.json, App/trivy-scan-image-results.json" // Attach the Trivy results
            )
        }
    }
}
