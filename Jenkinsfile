
pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Check Environment') {
            steps {
                sh '''
                    set -eu

                    echo "===== PHP ====="
                    php -v

                    echo "===== Composer ====="
                    composer --version

                    echo "===== Laravel Version ====="
                    grep -n '"laravel/framework"' composer.json
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    set -eu

                    composer validate --no-check-publish

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

                    if [ -d tests ]; then
                        php artisan test
                    else
                        echo "No tests directory found; skipping tests."
                    fi
                '''
            }
        }
    }

    post {
        success {
            echo 'Laravel CI pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed. Check Console Output.'
        }
    }
}
