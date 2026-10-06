pipeline {
    agent any

    environment {
        CI = 'true'
        // Injected secret credential
        API_TOKEN = credentials('sample-api-token')
    }

    stages {
        stage('Dependencies') {
            steps {
                sh 'npm ci'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test'
            }
        }

        stage('Security Check') {
            steps {
                // Verify credential injection and secret masking in logs
                sh 'echo "Verifying token delivery: $API_TOKEN"'
            }
        }
    }

    post {
        always {
            cleanWs deleteDirs: true, notFailBuild: true
        }
        failure {
            echo 'Pipeline failed. Check build logs for failure diagnostics.'
        }
    }
}