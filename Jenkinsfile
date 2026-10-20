
pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        PROD_HOST = '45.220.54.15'
        PROD_USER = 'laravel-deploy'
        PROD_DIR  = '/var/www/laravel'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Validate Project') {
            steps {
                sh '''
                    set -eu

                    php -v
                    composer --version
                    composer validate --no-check-publish
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    set -eu
                    composer install \
                        --no-interaction \
                        --prefer-dist \
                        --no-progress
                '''
            }
        }


            stage('Run Tests') {
                steps {
                    sh '''
                        set -eu

                        cp .env.example .env
                        php artisan key:generate

                        php artisan test
                    '''
                }
            }


        stage('Deploy Files') {
            steps {
                sh '''
                    set -eu

                    echo "Preparing Production directory..."

                    ssh -i /var/lib/jenkins/.ssh/id_ed25519 \
                        -o IdentitiesOnly=yes \
                        -o BatchMode=yes \
                        "$PROD_USER@$PROD_HOST" \
                        "mkdir -p '$PROD_DIR'"

                    echo "Copying Laravel files..."

                    rsync -az --delete \
                        --exclude='.env' \
                        --exclude='.git/' \
                        --exclude='storage/' \
                        --exclude='bootstrap/cache/' \
                        -e "ssh -i /var/lib/jenkins/.ssh/id_ed25519 -o IdentitiesOnly=yes -o BatchMode=yes" \
                        ./ "$PROD_USER@$PROD_HOST:$PROD_DIR/"

                    echo "Deployment files copied successfully."
                '''
            }
        }
    }

    post {
        success {
            echo 'CI and file deployment completed successfully.'
        }
        failure {
            echo 'Pipeline failed. Check Console Output.'
        }
    }
}
