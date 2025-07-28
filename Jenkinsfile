
pipeline {
    agent any
    
    // Environment variables
    environment {
        // Define any environment variables here
        PROJECT_NAME = 'my-First-Repo'
        GIT_REPO = 'https://github.com/2022f-mulbscs-048/my-First-Repo.git'
        BRANCH_NAME = "${env.BRANCH_NAME ?: 'main'}"
    }
    
    // Build options
    options {
        timestamps()
        timeout(time: 30, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
        disableConcurrentBuilds()
    }
    
    // Build triggers (optional)
    triggers {
        // Poll SCM every 5 minutes
        pollSCM('H/5 * * * *')
        // Or use webhook trigger
        // githubPush()
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo "Checking out code from ${GIT_REPO}"
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: "*/${BRANCH_NAME}"]],
                    extensions: [],
                    userRemoteConfigs: [[url: "${GIT_REPO}"]]
                ])
            }
        }
        
        stage('Detect Project Type') {
            steps {
                script {
                    // Detect project type based on files present
                    env.PROJECT_TYPE = detectProjectType()
                    echo "Detected project type: ${env.PROJECT_TYPE}"
                }
            }
        }
        
        stage('Install Dependencies') {
            steps {
                script {
                    switch(env.PROJECT_TYPE) {
                        case 'nodejs':
                            sh 'npm install'
                            break
                        case 'python':
                            sh '''
                                python3 -m venv venv
                                . venv/bin/activate
                                pip install -r requirements.txt || echo "No requirements.txt found"
                            '''
                            break
                        case 'java':
                            sh 'mvn clean install -DskipTests || gradle build -x test || echo "No build tool found"'
                            break
                        case 'dotnet':
                            sh 'dotnet restore'
                            break
                        default:
                            echo 'No specific dependencies to install'
                    }
                }
            }
        }
        
        stage('Code Quality') {
            parallel {
                stage('Linting') {
                    steps {
                        script {
                            switch(env.PROJECT_TYPE) {
                                case 'nodejs':
                                    sh 'npm run lint || echo "No lint script found"'
                                    break
                                case 'python':
                                    sh '''
                                        . venv/bin/activate || true
                                        pylint **/*.py || flake8 . || echo "No Python linter found"
                                    '''
                                    break
                                case 'java':
                                    sh 'mvn checkstyle:check || gradle checkstyleMain || echo "No checkstyle configured"'
                                    break
                                default:
                                    echo 'No linting configured for this project type'
                            }
                        }
                    }
                }
                
                stage('Security Scan') {
                    steps {
                        script {
                            // Run security scans based on project type
                            switch(env.PROJECT_TYPE) {
                                case 'nodejs':
                                    sh 'npm audit || echo "npm audit failed"'
                                    break
                                case 'python':
                                    sh '''
                                        . venv/bin/activate || true
                                        pip-audit || safety check || echo "No security scanner found"
                                    '''
                                    break
                                default:
                                    echo 'No security scan configured'
                            }
                        }
                    }
                }
            }
        }
        
        stage('Build') {
            steps {
                script {
                    switch(env.PROJECT_TYPE) {
                        case 'nodejs':
                            sh 'npm run build || echo "No build script found"'
                            break
                        case 'python':
                            sh '''
                                . venv/bin/activate || true
                                python -m py_compile **/*.py || echo "Python compilation check failed"
                            '''
                            break
                        case 'java':
                            sh 'mvn compile || gradle build || echo "Build failed"'
                            break
                        case 'dotnet':
                            sh 'dotnet build --configuration Release'
                            break
                        case 'static':
                            echo 'Static website - no build required'
                            break
                        default:
                            echo 'No build step required'
                    }
                }
            }
        }
        
        stage('Test') {
            steps {
                script {
                    switch(env.PROJECT_TYPE) {
                        case 'nodejs':
                            sh 'npm test || echo "No tests found"'
                            break
                        case 'python':
                            sh '''
                                . venv/bin/activate || true
                                pytest || python -m unittest discover || echo "No tests found"
                            '''
                            break
                        case 'java':
                            sh 'mvn test || gradle test || echo "No tests found"'
                            break
                        case 'dotnet':
                            sh 'dotnet test'
                            break
                        default:
                            echo 'No tests configured'
                    }
                }
            }
            post {
                always {
                    // Publish test results if available
                    junit allowEmptyResults: true, testResults: '**/target/surefire-reports/*.xml, **/build/test-results/test/*.xml, **/test-results/*.xml'
                }
            }
        }
        
        stage('Package') {
            when {
                expression { env.BRANCH_NAME == 'main' || env.BRANCH_NAME == 'master' }
            }
            steps {
                script {
                    switch(env.PROJECT_TYPE) {
                        case 'nodejs':
                            sh 'npm pack || echo "Packaging failed"'
                            break
                        case 'python':
                            sh '''
                                . venv/bin/activate || true
                                python setup.py sdist bdist_wheel || echo "No setup.py found"
                            '''
                            break
                        case 'java':
                            sh 'mvn package || gradle jar || echo "Packaging failed"'
                            break
                        case 'dotnet':
                            sh 'dotnet publish -c Release -o ./publish'
                            break
                        case 'docker':
                            sh '''
                                docker build -t ${PROJECT_NAME}:${BUILD_NUMBER} .
                                docker tag ${PROJECT_NAME}:${BUILD_NUMBER} ${PROJECT_NAME}:latest
                            '''
                            break
                        default:
                            echo 'No packaging configured'
                    }
                }
            }
        }
        
        stage('Archive Artifacts') {
            when {
                expression { env.BRANCH_NAME == 'main' || env.BRANCH_NAME == 'master' }
            }
            steps {
                script {
                    switch(env.PROJECT_TYPE) {
                        case 'nodejs':
                            archiveArtifacts artifacts: '*.tgz, dist/**/*', allowEmptyArchive: true
                            break
                        case 'python':
                            archiveArtifacts artifacts: 'dist/*', allowEmptyArchive: true
                            break
                        case 'java':
                            archiveArtifacts artifacts: '**/target/*.jar, **/build/libs/*.jar', allowEmptyArchive: true
                            break
                        case 'dotnet':
                            archiveArtifacts artifacts: 'publish/**/*', allowEmptyArchive: true
                            break
                        default:
                            archiveArtifacts artifacts: '**/*', excludes: '**/node_modules/**, **/.git/**, **/venv/**, **/target/**, **/build/**', allowEmptyArchive: true
                    }
                }
            }
        }
        
        stage('Deploy') {
            when {
                expression { env.BRANCH_NAME == 'main' || env.BRANCH_NAME == 'master' }
            }
            steps {
                script {
                    echo "Deployment stage - configure based on your deployment target"
                    // Add deployment steps here based on your infrastructure
                    // Examples:
                    // - Deploy to AWS S3 for static sites
                    // - Push Docker image to registry
                    // - Deploy to Kubernetes
                    // - Upload to artifact repository (Nexus, Artifactory)
                    // - Deploy to application server
                }
            }
        }
    }
    
    post {
        always {
            echo 'Pipeline execution completed'
            cleanWs()
        }
        success {
            echo 'Pipeline succeeded!'
            // Send success notification
            // slackSend(color: 'good', message: "Build Successful: ${env.JOB_NAME} - ${env.BUILD_NUMBER}")
        }
        failure {
            echo 'Pipeline failed!'
            // Send failure notification
            // slackSend(color: 'danger', message: "Build Failed: ${env.JOB_NAME} - ${env.BUILD_NUMBER}")
        }
        unstable {
            echo 'Pipeline is unstable'
            // Send unstable notification
        }
    }
}

// Helper function to detect project type
def detectProjectType() {
    if (fileExists('package.json')) {
        return 'nodejs'
    } else if (fileExists('requirements.txt') || fileExists('setup.py') || fileExists('pyproject.toml')) {
        return 'python'
    } else if (fileExists('pom.xml') || fileExists('build.gradle') || fileExists('build.gradle.kts')) {
        return 'java'
    } else if (fileExists('*.csproj') || fileExists('*.sln')) {
        return 'dotnet'
    } else if (fileExists('Dockerfile')) {
        return 'docker'
    } else if (fileExists('index.html')) {
        return 'static'
    } else {
        return 'unknown'
    }
}
