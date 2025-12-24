pipeline {
    agent any

    stages {

        stage("Initial cleanup") {
            steps {
                dir("${WORKSPACE}") {
                    deleteDir()
                }
            }
        }

        stage('Checkout SCM') {
            steps {
                git branch: 'main', url: 'https://github.com/Munyat/php-todo.git'
            }
        }

        stage('Prepare Dependencies') {
            steps {
                dir("${WORKSPACE}") {
                    // Copy environment file
                    sh 'cp .env.sample .env'

                    // Ensure necessary directories exist with proper permissions
                    sh '''
                        mkdir -p bootstrap/cache storage
                        chmod -R 775 bootstrap/cache storage
                        chown -R $(whoami):$(whoami) bootstrap/cache storage
                    '''

                    // Install dependencies
                    sh 'composer install --no-interaction --prefer-dist'

                    // Generate app key first
                    sh 'php artisan key:generate'

                    // Clear any cached config, routes, views
                    sh '''
                        php artisan config:clear
                        php artisan cache:clear
                        php artisan route:clear
                        php artisan view:clear
                    '''

                    // Run migrations and seed database
                    sh 'php artisan migrate --force'
                    sh 'php artisan db:seed --force'
                }
            }
        }
    }
}
