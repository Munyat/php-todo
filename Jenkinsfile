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
                # Ensure we have write permissions
                mkdir -p build/logs
                chmod -R 755 build
                
                # Run phploc
                echo "=== Running phploc analysis ==="
                phploc app/ --log-csv build/logs/phploc.csv
                
                # Verify CSV was created
                echo "=== Verifying CSV file ==="
                ls -la build/logs/phploc.csv
                echo "=== First 3 lines of CSV ==="
                head -3 build/logs/phploc.csv
            '''
        }
    }
}
stage('Plot Code Coverage Report') {
    steps {
        plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Lines of Code (LOC),Comment Lines of Code (CLOC),Non-Comment Lines of Code (NCLOC),Logical Lines of Code (LLOC)                          ', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'A - Lines of code', yaxis: 'Lines of Code'
        plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Directories,Files,Namespaces', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'B - Structures Containers', yaxis: 'Count'
        plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Average Class Length (LLOC),Average Method Length (LLOC),Average Function Length (LLOC)', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'C - Average Length', yaxis: 'Average Lines of Code'
        plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Cyclomatic Complexity / Lines of Code,Cyclomatic Complexity / Number of Methods ', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'D - Relative Cyclomatic Complexity', yaxis: 'Cyclomatic Complexity by Structure'      
        plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Classes,Abstract Classes,Concrete Classes', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'E - Types of Classes', yaxis: 'Count'
        plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Methods,Non-Static Methods,Static Methods,Public Methods,Non-Public Methods', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'F - Types of Methods', yaxis: 'Count'
        plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Constants,Global Constants,Class Constants', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'G - Types of Constants', yaxis: 'Count'
        plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Test Classes,Test Methods', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'I - Testing', yaxis: 'Count'
        plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Logical Lines of Code (LLOC),Classes Length (LLOC),Functions Length (LLOC),LLOC outside functions or classes ', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'AB - Code Structure by Logical Lines of Code', yaxis: 'Logical Lines of Code'
        plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Functions,Named Functions,Anonymous Functions', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'H - Types of Functions', yaxis: 'Count'
        plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Interfaces,Traits,Classes,Methods,Functions,Constants', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'BB - Structure Objects', yaxis: 'Count'
    }
}

stage('Package Artifact') {
    steps {
        dir("${WORKSPACE}") {
            sh 'zip -qr php-todo.zip . -x "*.git*" "node_modules/*"'
        }
    }
}
stage('Upload Artifact to Artifactory') {
    steps {
        script {
            def server = Artifactory.server 'artifactory-server'
            def uploadSpec = """{
                "files": [
                    {
                        "pattern": "php-todo.zip",
                        "target": "php-todo-repo/php-todo/",
                        "props": "type=zip;status=ready"
                    }
                ]
            }"""
            server.upload spec: uploadSpec
        }
    }
}

stage('Deploy to Dev Environment') {
    steps {
        build job: 'ansible-config-mgt/main',
              parameters: [[$class: 'StringParameterValue', name: 'env', value: 'dev']],
              propagate: false,
              wait: true
    }
}
        stage('SonarQube Quality Gate') {
            // Only run for specific branches
            when { 
                branch pattern: "^develop*|^hotfix*|^release*|^main*|^master*", 
                comparator: "REGEXP"
            }
            
            steps {
                // Run SonarQube analysis
                withSonarQubeEnv('sonarqube') {
                    sh """
                        ${scannerHome}/bin/sonar-scanner \
                        -Dproject.settings=sonar-project.properties \
                        -Dsonar.projectBaseDir=${WORKSPACE}
                    """
                }
                
                // Wait for Quality Gate result (max 1 minute)
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
            
            post {
                success {
                    echo 'Quality Gate passed!'
                }
                failure {
                    echo 'Quality Gate failed! Check SonarQube for details.'
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