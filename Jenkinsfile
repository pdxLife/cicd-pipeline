pipeline {
    agent any
    
    triggers {
        // Poll SCM every minute for changes
        pollSCM('* * * * *')
        // Or use GitHub webhook trigger (configure in Jenkins job settings)
        // githubPush()
    }

    parameters {
        string(name: 'IMAGE_REPO', defaultValue: 'bekzhanov/cicd-pipeline', description: 'repo and name')
        string(name: 'REGISTRY_CREDENTIALS_ID', defaultValue: 'docker_registry_creds', description: 'Jenkins creds ID')
    }

    stages {
        stage('GIt Checkout') {
            steps {
                echo '📦 Checking out source code from GitHub...'
                checkout scm
                
                sh '''
                    echo "Repository checked out successfully"
                    echo "Current branch: $(git branch --show-current || echo 'detached HEAD')"
                    echo "Commit: $(git rev-parse --short HEAD)"
                    echo ""
                    echo "Project structure:"
                    ls -la
                '''
            }
        }

        stage('Application Build') {
            steps {
                sh 'chmod +x scripts/build.sh'
                sh 'scripts/build.sh'
            }
        }

        stage('Tests') {
            steps {
                sh 'chmod +x scripts/test.sh'
                sh 'scripts/test.sh'
            }
        }
        
        stage('Docker image - Build') {
            steps {
                script {
                env.IMAGE_TAG  = "${IMAGE_REPO}:${env.BUILD_NUMBER}"
                env.IMAGE_LATEST = "${IMAGE_REPO}:latest"

                sh "docker build -t ${env.IMAGE_TAG} ."
                sh "docker tag ${env.IMAGE_TAG} ${env.IMAGE_LATEST}"
                }
            }
        }

        stage('Docker image - Push') {
            steps {
                echo 'Starting to build docker image'
                withCredentials([usernamePassword(
                    credentialsId: env.REGISTRY_CREDENTIALS_ID,
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_TOKEN'
                )]) {
                    sh '''
                        set -e
                        echo "$DOCKER_TOKEN" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push "$IMAGE_TAG"
                        docker push "$IMAGE_LATEST"
                    '''
                 }
       	     }
   	 }
    }
    
    post {
        success {
            echo '✅ Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed!'
        }
        always {
            sh 'docker logout || true'
            sh 'docker image prune -f || true'
            echo "Image pruned"
        }
    }
}
