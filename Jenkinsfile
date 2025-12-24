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
            // Step 1: Copy environment file
            sh 'cp .env.sample .env'
            
            // Step 2: Generate encryption key FIRST
            sh 'php artisan key:generate'
            
            // Step 3: Create necessary directories
            sh '''
                mkdir -p bootstrap/cache
                chmod 775 bootstrap/cache
                mkdir -p storage/framework/{sessions,views,cache}
                mkdir -p storage/logs
                chmod -R 775 storage
            '''
            
            // Step 4: Install dependencies (without scripts)
            sh 'composer install --no-interaction --prefer-dist --no-scripts'
            
            // Step 5: Run Laravel optimization commands
            sh '''
                php artisan config:clear
                php artisan cache:clear
                php artisan route:clear
                php artisan view:clear
                php artisan optimize
            '''
            
            // Step 6: Setup database
            sh 'php artisan migrate --force'
            sh 'php artisan db:seed --force'
        }
    }
}
stage('Verify Setup') {
    steps {
        dir("${WORKSPACE}") {
            sh '''
                echo "=== Verifying Environment ==="
                
                # Check if APP_KEY exists and is valid
                if grep -q "APP_KEY=base64:" .env; then
                    echo "✓ APP_KEY is properly set"
                else
                    echo "✗ ERROR: APP_KEY is missing or invalid"
                    exit 1
                fi
                
                # Check database connection
                echo "Testing database connection..."
                php artisan tinker --execute="
                    try {
                        \\DB::connection()->getPdo();
                        echo '✓ Database connected successfully';
                    } catch (\\Exception \$e) {
                        echo '✗ Database connection failed: ' . \$e->getMessage();
                        exit(1);
                    }
                "
            '''
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