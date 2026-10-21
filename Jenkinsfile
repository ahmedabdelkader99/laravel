
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



stage('Verify Production Environment') {
    steps {
        sh '''
            set -eu

            ssh -i /var/lib/jenkins/.ssh/id_ed25519 \
                -o IdentitiesOnly=yes \
                -o BatchMode=yes \
                "$PROD_USER@$PROD_HOST" \
                "set -eu
                 cd '$PROD_DIR'

                 echo 'Checking Laravel files...'
                 test -f artisan
                 test -f public/index.php
                 test -f vendor/autoload.php

                 echo 'Checking production .env...'
                 test -f .env

                 echo 'Checking PHP version...'
                 php -r 'exit(PHP_VERSION_ID >= 80300 ? 0 : 1);' || {
                     echo 'ERROR: Laravel 13 requires PHP 8.3 or newer.'
                     php -v
                     exit 1
                 }

                 echo 'Checking MariaDB...'
                 systemctl is-active --quiet mariadb

                 echo 'Production prerequisite checks passed.'
                "
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
