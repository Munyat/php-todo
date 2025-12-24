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
                    
                    // Create cache directory
                    sh '''
                        mkdir -p bootstrap/cache
                        chmod 775 bootstrap/cache
                    '''
                    
                    // Install dependencies WITHOUT scripts
                    sh 'composer install --no-interaction --prefer-dist --no-scripts'
                    
                    // FIX: Don't create empty cache files. Let Laravel create them properly.
                    // FIX: Run clear-compiled with exception handling
                    sh '''
                        # Try to clear compiled, ignore errors if it fails
                        php artisan clear-compiled || true
                        php artisan optimize || true
                    '''
                    
                    // Generate app key
                    sh 'php artisan key:generate'
                    
                    // Clear any cached config, routes, views
                    sh '''
                        php artisan config:clear
                        php artisan cache:clear
                        php artisan route:clear
                        php artisan view:clear
                    '''
                    
                    // Create storage directories and set permissions
                    sh '''
                        mkdir -p storage/framework/sessions
                        mkdir -p storage/framework/views
                        mkdir -p storage/framework/cache
                        mkdir -p storage/logs
                        chmod -R 775 storage
                    '''
                    
                    // Run migrations and seed database
                    sh 'php artisan migrate --force'
                    sh 'php artisan db:seed --force'
                }
            }
        }
        stage('Execute Unit Tests') {
            steps {
              sh './vendor/bin/phpunit'
            } 
        }
    }
}