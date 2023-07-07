pipeline {
    agent any 
    stages {
        stage('Build Backend') {
            steps {
                bat 'mvn clean package -DskipTests=true'
            }
        }
        stage('Unit Tests') {
            steps {
                bat 'mvn test'
            }
        }
        stage('Sonar Analysis') {
            environment {
                scannerHome = tool 'SONAR_SCANNER'
            }
            steps {
                withSonarQubeEnv('SONAR_LOCAL'){
                    bat "${scannerHome}/bin/sonar-scanner -e -Dsonar.projectKey=DeployBack -Dsonar.host.url=http://localhost:9000 -Dsonar.login=9c925efbe4aa3823f5919cbd246d16877bdc866a -Dsonar.java.binaries=target -Dsonar.coverage.exclusions=**/.mvn/**,**/src/test/**,**/model/**,**Application.java"
                }
            }
        }
        stage('Quality Gate') {
            steps {
                sleep(5)
                timeout(time: 1, unit: 'MINUTES'){
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Deploy Backend'){
            steps {                
                    deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks-backend', war: 'target/tasks-backend.war'
            }
        }
        stage('API Test'){
            steps {
                dir('api-test') {
                    git credentialsId: 'github_login', url: 'https://github.com/RicardoJustino/tasks-api-test'
                    bat 'mvn test'
                }
            }
        }
        stage('Deploy Frontend'){
            steps {
                dir('frontend') {
                    git credentialsId: 'github_login', url: 'https://github.com/RicardoJustino/tasks-frontend'
                    bat 'mvn clean package'
                    deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks', war: 'target/tasks.war'
                }
            }
        }
        stage('functional Test'){
            steps {
                dir('functional-test') {
                    git credentialsId: 'github_login', url: 'https://github.com/RicardoJustino/tasks-funcional-tests'
                    bat 'mvn test'
                }
            }
        }
        stage('Deploy Prod'){
            steps {
                bat 'docker-compose build'
                bat 'docker-compose up -d'
            }
        }
    }
}