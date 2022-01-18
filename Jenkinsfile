pipeline{
    agent any
    stages{
        stage('Build Backend'){
            steps{
                bat 'mvn clean package -DskipTests=true'
            }
        }
        stage('Unit Tests'){
            steps{
                bat 'mvn test'
            }
        }
        // stage('Sonar Analysis'){
        //     environment{
        //         scannerHome = tool 'SONAR_SCANNER'
        //     }
        //     steps{
        //         withSonarQubeEnv('SONAR_LOCAL'){
        //             bat "${scannerHome}/bin/sonar-scanner -e -Dsonar.projectKey=DeployBackend -Dsonar.host.url=http://localhost:9000 -Dsonar.login=7822f529ac82a6abe88017b6324eefdc24a714f3 -Dsonar.java.binaries=target -Dsonar.coverage.exclusions=**/.mvn/**,**/src/test/**,**/model/**,**Application.java"
        //         }
        //     }
        // }
        // stage('Quality Gate'){
        //     steps{
        //         sleep(30)
        //         timeout(time: 1, unit: 'MINUTES'){
        //             waitForQualityGate abortPipeline: true
        //         }                
        //     }
        // }
        stage('Deploy Backend'){
            steps{
                deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks-backend', war: 'target/tasks-backend.war'
            }
        }
        stage('API Test'){
            steps{
                dir('api-test') {
                    git credentialsId: 'github_login', url: 'https://github.com/samuelssaquino/tasks-api-test'
                    bat 'mvn test'
                }                
            }
        }
        stage('Deploy Frontend'){
            steps{
                dir('frontend'){
                    git credentialsId: 'github_login', url: 'https://github.com/samuelssaquino/tasks-frontend'
                    bat 'mvn clean package'
                    deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks', war: 'target/tasks.war'
                }                
            }
        }
        stage('Functional Test Web'){
            steps{
                dir('functional-test-web') {
                    git credentialsId: 'github_login', url: 'https://github.com/samuelssaquino/tasks-functional-tests-web'
                    bat 'mvn test'
                }                
            }
        }
        stage('Deploy Prod'){
            steps{
                bat 'docker-compose build'
                bat 'docker-compose up -d'
            }
        }
        stage('Health Check'){
            steps{
                sleep(5)
                dir('functional-test-web') {         
                    bat 'mvn verify -Dskip.surefire.tests'
                }                
            }
        }
    }
    post{
        always{
            junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml, api-test/target/surefire-reports/*.xml, functional-test-web/target/surefire-reports/*.xml, functional-test-web/target/failsafe-reports/*.xml'
        }

    }
}



