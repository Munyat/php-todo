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
                    
                    // FIX: Create cache directory and files BEFORE composer install
                    sh '''
                        mkdir -p bootstrap/cache
                        chmod 775 bootstrap/cache
                        # Create the files that artisan clear-compiled expects
                        touch bootstrap/cache/services.php
                        touch bootstrap/cache/packages.php
                        touch bootstrap/cache/compiled.php
                        chmod 664 bootstrap/cache/*.php
                    '''
                    
                    // FIX: Install dependencies WITHOUT running scripts first
                    sh 'composer install --no-interaction --prefer-dist --no-scripts'
                    
                    // FIX: Now run the artisan commands manually
                    sh '''
                        php artisan clear-compiled
                        php artisan optimize
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
    }
}