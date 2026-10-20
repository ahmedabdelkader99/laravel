
pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
        skipDefaultCheckout(true)
    }

    environment {
        APP_DIR = '/var/www/laravel'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                sh '''
                    echo "Repository checked out successfully."
                    echo "Current branch: $(git branch --show-current)"
                    test -f composer.json || {
                        echo "ERROR: composer.json not found."
                        exit 1
                    }
                    test -f artisan || {
                        echo "ERROR: Laravel artisan file not found."
                        exit 1
                    }
                '''
            }
        }

        stage('Check Build Environment') {
            steps {
                sh '''
                    set -eu

                    echo "===== PHP ====="
                    php -v

                    echo "===== Composer ====="
                    composer --version

                    echo "===== Laravel Requirements ====="
                    php -r '
                        $data = json_decode(
                            file_get_contents("composer.json"),
                            true,
                            512,
                            JSON_THROW_ON_ERROR
                        );
                        echo "PHP requirement: ",
                            $data["require"]["php"] ?? "not specified",
                            PHP_EOL;
                        echo "Laravel framework: ",
                            $data["require"]["laravel/framework"] ?? "not specified",
                            PHP_EOL;
                    '

                    echo "===== Validate Composer Configuration ====="
                    composer validate --no-check-publish
                '''
            }
        }
    }

    post {
        success {
            echo 'Validation completed successfully. Production was not changed.'
        }
        failure {
            echo 'Validation failed. Check the Console Output before proceeding.'
        }
    }
}
