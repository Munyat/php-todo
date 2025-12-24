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
                    // 1. Copy environment
                    sh 'cp .env.sample .env'
                    
                    // 2. Create directories individually
                    sh '''
                        # Create cache directory
                        mkdir -p bootstrap/cache
                        chmod 775 bootstrap/cache
                        
                        # Create storage directories one by one
                        mkdir -p storage/framework/sessions
                        mkdir -p storage/framework/views
                        mkdir -p storage/framework/cache
                        mkdir -p storage/logs
                        
                        # Set permissions
                        chmod -R 775 storage
                        chmod 777 storage/framework/sessions
                    '''
                    
                    // 3. Install dependencies
                    sh 'composer install --no-interaction --prefer-dist --no-scripts'
                    
                    // 4. Generate key
                    sh 'php artisan key:generate'
                    
                    // 5. Clear caches
                    sh '''
                        php artisan config:clear
                        php artisan cache:clear
                        php artisan route:clear
                        php artisan view:clear
                    '''
                    
                    // 6. Setup database
                    sh 'php artisan migrate --force'
                    sh 'php artisan db:seed --force'
                }
            }
        }

        stage('Execute Unit Tests') {
            steps {
                dir("${WORKSPACE}") {
                    sh './vendor/bin/phpunit'
                }
            }
        }
stage('Code Analysis') {
    steps {
        dir("${WORKSPACE}") {
            sh '''
                mkdir -p build/logs
                phploc app/ --log-csv build/logs/phploc.csv
            '''
        }
    }
}        
    }
    
    post {
        always {
            echo "Build completed - see test results above"
        }
        success {
            echo "✓ Pipeline succeeded!"
        }
        failure {
            echo "✗ Pipeline failed"
        }
    }
}