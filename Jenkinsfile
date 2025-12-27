pipeline {
    agent any
    
    triggers {
        // Poll SCM every minute for changes
        pollSCM('* * * * *')
        // Or use GitHub webhook trigger (configure in Jenkins job settings)
        // githubPush()
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
        
        stage('Docker image build') {
            steps {
                echo 'Starting to build docker image'

                script {
                    def customImage = docker.build("my-image:${env.BUILD_ID}")
                    customImage.push()
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
    }

}
