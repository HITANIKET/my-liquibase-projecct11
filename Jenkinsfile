pipeline {
    agent any
    
    environment {
        DB_URL = 'jdbc:oracle:thin:@45.114.142.57:1525/YIPDV'
        DB_USERNAME = credentials('YIPBL')
        DB_PASSWORD = credentials('YIPBL')
        MAVEN_HOME = tool 'Maven'
        PATH = "${MAVEN_HOME}/bin:${PATH}"
    }
    
    tools {
        maven 'Maven'
        jdk 'JDK11'
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out code from GitHub...'
                checkout scm
            }
        }
        
        stage('Validate') {
            steps {
                echo 'Validating project structure...'
                sh 'mvn validate'
            }
        }
        
        stage('Compile') {
            steps {
                echo 'Compiling the project...'
                sh 'mvn clean compile'
            }
        }
        
        stage('Test Connection') {
            steps {
                echo 'Testing database connection...'
                script {
                    try {
                        sh '''
                            mvn liquibase:status \
                                -Dliquibase.url=${DB_URL} \
                                -Dliquibase.username=${DB_USERNAME} \
                                -Dliquibase.password=${DB_PASSWORD}
                        '''
                        echo 'Database connection successful'
                    } catch (Exception e) {
                        error "Database connection failed: ${e.getMessage()}"
                    }
                }
            }
        }
        
        stage('Liquibase Update') {
            steps {
                echo 'Running Liquibase database migrations...'
                sh '''
                    mvn liquibase:update \
                        -Dliquibase.url=${DB_URL} \
                        -Dliquibase.username=${DB_USERNAME} \
                        -Dliquibase.password=${DB_PASSWORD}
                '''
            }
        }
        
        stage('Generate Changelog Report') {
            steps {
                echo 'Generating changelog report...'
                sh '''
                    mvn liquibase:status \
                        -Dliquibase.url=${DB_URL} \
                        -Dliquibase.username=${DB_USERNAME} \
                        -Dliquibase.password=${DB_PASSWORD} \
                        -Dliquibase.verbose=true
                '''
            }
        }
    }
    
    post {
        always {
            echo 'Pipeline execution completed'
            archiveArtifacts artifacts: 'target/**/*', fingerprint: true
        }
        success {
            echo 'Database migration completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check the logs for details.'
        }
    }
}