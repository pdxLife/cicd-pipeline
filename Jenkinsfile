pipeline {
    agent any
    
    triggers {
        // Poll SCM every minute for changes
        pollSCM('* * * * *')
        // Or use GitHub webhook trigger (configure in Jenkins job settings)
        // githubPush()
    }

    parameters {
        string(name: 'REGISTRY_HOST', defaultValue: 'docker.io', description: 'Registry hostname (blank = Docker Hub)')
        string(name: 'DOCKER_IMAGE_REPO', defaultValue: 'bekzhanov/cicd-pipeline', description: 'Repository and name')
        string(name: 'REGISTRY_CREDENTIALS_ID', defaultValue: 'docker_registry_creds', description: 'Jenkins creds ID (username/password)')
    }

    environment {
        DOCKER_HUB_HOST = 'docker.io'
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
                def host = (params.REGISTRY_HOST ?: '').trim()
                def repo = params.DOCKER_IMAGE_REPO.trim()

                // Fully qualified image name:
                // - Docker Hub: repo only, like "youruser/app"
                // - Other registries: "host/repo"
                def imageBase = host ? "${host}/${repo}" : repo

                env.IMAGE_BASE = imageBase
                env.IMAGE_TAG  = "${imageBase}:${env.BUILD_NUMBER}"
                env.IMAGE_LATEST = "${imageBase}:latest"

                sh "docker build -t ${env.IMAGE_TAG} ."
                sh "docker tag ${env.IMAGE_TAG} ${env.IMAGE_LATEST}"
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
