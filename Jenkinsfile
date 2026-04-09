pipeline {
    agent { label 'ansible-agent' }

    environment {
        DEPLOY_HOST = '178.128.93.188'
        DEPLOY_PATH = '/var/www/html/tek_chansetha'
        DEPLOY_USER = 'root'
    }

    stages {
        stage('Checkout') {
            steps { checkout scm }
        }

        stage('Build') {
            steps {
                sh '''
                    composer install --no-interaction --prefer-dist

                    # create .env if not exists
                    cp .env.example .env || true

                    php artisan key:generate --force

                    npm install
                    npm run build
                '''
            }
        }

        stage('Test') {
            steps {
                sh 'php artisan test'
            }
        }

        stage('Deploy') {
            when { branch 'main' }
            steps {
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'deploy-ssh-key',
                    keyFileVariable: 'SSH_KEY'
                )]) {
                    sh '''
                        ansible-playbook deploy.yml \
                          -i "${DEPLOY_HOST}," \
                          -u ${DEPLOY_USER} \
                          --private-key "$SSH_KEY" \
                          --extra-vars "deploy_path=${DEPLOY_PATH}"
                    '''
                }
            }
        }
    }

    post {
        failure {
            // mail to: 'chansethatek@gmail.com',
            //      subject: "FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            //      body: "Build failed!\nURL: ${env.BUILD_URL}"

            withCredentials([
                string(credentialsId: 'telegram-token',   variable: 'TG_TOKEN'),
                string(credentialsId: 'telegram-chat-id', variable: 'TG_CHAT')
            ]) {
                sh """
                    curl -s -X POST \
                      "https://api.telegram.org/bot\${TG_TOKEN}/sendMessage" \
                      -d chat_id="\${TG_CHAT}" \
                      -d text="FAILED: ${env.JOB_NAME} %23${env.BUILD_NUMBER}%0A${env.BUILD_URL}"
                """
            }
        }
        success {
            echo "Deployed to http://${DEPLOY_HOST}/tek_chansetha"
            
            withCredentials([
                string(credentialsId: 'telegram-token',   variable: 'TG_TOKEN'),
                string(credentialsId: 'telegram-chat-id', variable: 'TG_CHAT')
            ])sh """
                    curl -s -X POST \
                      "https://api.telegram.org/bot\${TG_TOKEN}/sendMessage" \
                      -d chat_id="\${TG_CHAT}" \
                      -d text="SUCCEEDED: ${env.JOB_NAME} %23${env.BUILD_NUMBER}%0A${env.BUILD_URL}"
                """
        }
    }
}
