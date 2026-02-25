pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', 
                    url: 'git@github.com:malikhamza-v/stokse-container.git', 
                    credentialsId: 'git-ssh-key'
            }
        }

        stage('Cleanup Old Container') {
             steps {
                 script {
                     sh "docker compose down"
                 }
             }
        }

        stage('Create Env File') {
            steps {
                withCredentials([file(credentialsId: 'stokse-db-variable-env', variable: 'ENV_FILE')]) {
                    script {
                        def envContent = readFile(env: ENV_FILE)
                        writeFile file: 'db_variables.env', text: envContent
                    }
                }
            }
        }

        stage('Deploy with Docker Compose') {
            steps {
                script {
                    // Compose will recreate the container automatically
                    sh "docker compose up -d --build --remove-orphans"
                }
            }
        }
        
        stage('Verify') {
            steps {
                sh "docker compose ps"
            }
        }
    }

    post {
        success {
            echo 'Deployment completed successfully!'
        }
        failure {
            echo 'Deployment failed. Check the logs!'
        }
    }
}