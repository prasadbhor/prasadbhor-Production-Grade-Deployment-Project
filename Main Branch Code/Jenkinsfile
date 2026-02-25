pipeline {
    agent any

    options {
        disableConcurrentBuilds() // prevent multiple builds at once
        skipDefaultCheckout()      // We'll explicitly checkout
    }

    environment {
        IMAGE_NAME = "prasadb25/multibranch-flask-app"
        GIT_USER   = "prasadbhor"
        GIT_EMAIL  = "prasadbhor0767@gmail.com"
    }

    stages {

        stage('Checkout') {
            steps {
                // Explicit checkout using the branch being built
                checkout([$class: 'GitSCM', branches: [[name: env.BRANCH_NAME]],
                          userRemoteConfigs: [[url: 'https://github.com/prasadbhor/prasadbhor-Production-Grade-Deployment-Project.git']]])
            }
        }

        stage('Build and Push Docker Image') {
            when { branch 'main' }  // Only on main branch
            steps {
                script {
                    env.IMAGE_TAG = "build-${BUILD_NUMBER}"

                    withCredentials([usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh """
                        set -e
                        docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                        """
                    }
                }
            }
        }

        stage('Update K8s Deployment') {
            when { branch 'main' }  // Only on main branch
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'github-creds',
                        usernameVariable: 'GIT_USERNAME',
                        passwordVariable: 'GIT_TOKEN'
                    )]) {
                        sh """
                        set -e
                        git config user.name "$GIT_USER"
                        git config user.email "$GIT_EMAIL"

                        git fetch origin
                        git checkout main
                        git reset --hard origin/main

                        # Update the image in Kubernetes deployment YAML
                        sed -i "s|image:.*|image: ${IMAGE_NAME}:${IMAGE_TAG}|" k8s/deployment.yml

                        git add k8s/deployment.yml
                        # Commit only if there is a change
                        git diff --cached --quiet || git commit -m "Updated image to ${IMAGE_TAG}"
                        git push https://${GIT_USERNAME}:${GIT_TOKEN}@github.com/prasadbhor/prasadbhor-Production-Grade-Deployment-Project.git main
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Build #${BUILD_NUMBER} finished successfully!"
        }
        failure {
            echo "Build #${BUILD_NUMBER} failed!"
        }
    }
}}