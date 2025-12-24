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
                    
                    // 2. Create directories
                    sh '''
                        mkdir -p bootstrap/cache
                        chmod 775 bootstrap/cache
                        mkdir -p storage/framework/{sessions,views,cache}
                        mkdir -p storage/logs
                        chmod -R 775 storage
                    '''
                    
                    // 3. Install dependencies
                    sh 'composer install --no-interaction --prefer-dist --no-scripts'
                    
                    // 4. Generate key
                    sh 'php artisan key:generate'
                    
                    // 5. Clear caches (skip optimize or handle error)
                    sh '''
                        php artisan config:clear
                        php artisan cache:clear
                        php artisan route:clear
                        php artisan view:clear
                        # Skip optimize - it's causing issues but not needed
                        # php artisan optimize || true
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