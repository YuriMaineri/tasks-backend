pipeline {
    agent any
    stages{
        stage('Build Backend') {
            steps {
                sh 'mvn clean package -DskipTests=true'
            }
        }
        stage('Unit Tests') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Sonar Analysis') {
            environment {
                scannerHome = tool 'SONAR_SCANNER'
            }
            steps {
                withSonarQubeEnv('SONAR_LOCAL') {
                    sh "${scannerHome}/bin/sonar-scanner -e -Dsonar.projectKey=DeployBack -Dsonar.host.url=http://localhost:9000 -Dsonar.login=9aa0c451a1c307c2eb1254fb7bf22cb27edbd232 -Dsonar.java.binaries=target -Dsonar.coverage.exclusions=**/.mvn/**,**/src/test/**,**/model/**,**Application.java"
                }
            }
        }
        stage('Quality Gate') {
            steps{
                sleep(10)
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Deploy Backend') {
            steps{
                deploy adapters: [tomcat8(credentialsId: 'tomcat_login', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks-backend', war: 'target/tasks-backend.war'
            }
        }
        stage('API Test') {
            steps{
                dir('api-test') {
                    git 'https://github.com/YuriMaineri/task-api-test'                
                    sh 'mvn test'
                }
            }
        }
        stage('Deploy Frontend') {
            steps{
                dir('frontend'){
                    git 'https://github.com/YuriMaineri/tasks-frontend'    
                    sh 'mvn clean package'            
                    deploy adapters: [tomcat8(credentialsId: 'tomcat_login', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks', war: 'target/tasks.war'
                }
            }
        }
        stage('Functional Test') {
            steps{
                dir('functional-test') {
                    git 'https://github.com/YuriMaineri/tasks-functional-test'                
                    sh 'mvn test'
                }
            }
        }
        stage('Deploy Prod') {
            steps{
                sh 'docker-compose build'
                sh 'docker-compose up -d'
            }
        }
        stage('Teste Producao') {
            steps{
                sleep(10)
                dir('functional-test') {                
                    sh 'mvn verify -Dskip.surefire.tests'
                }
            }
        }
    }
    post {
        always {
            junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml, api-test/target/surefire-reports;*.xml, functional-test/target/surefire-reports/*.xml, functional-test/target/failsafe-reports/*.xml'
        }
        unsuccessful {
            emailext attachLog: true, body: 'Log esta em anexo', subject: 'Build $BUILD_NUMBER has failed', to: 'yurimaineri5@gmail.com'
        }
        fixed {
            emailext attachLog: true, body: 'Log esta em anexo', subject: 'Build $BUILD_NUMBER is fine', to: 'yurimaineri5@gmail.com'
        }
    }
}
