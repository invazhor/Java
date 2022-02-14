def COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger',
]
pipeline {
    agent any
    //Set up local variables for your pipeline
    environment {
        //test variable: 0=success, 1=fail; must be string
        doError = '0'
        BUILD_USER = ''
    }
    //tools {
    //    maven 'Maven'
    //    nodejs 'NodeJs'
    //}
  
    stages {
        stage('initial'){
            steps{
             figlet 'Iniciando...'
             sh '''
              echo "PATH = ${PATH}"
              echo "M2_HOME = ${M2_HOME}"
              '''
            }
        }
    
        stage('SCA'){
            steps{
                figlet 'Dependency-Check SCA'
                sh 'mvn org.owasp:dependency-check-maven:check'
                
                archiveArtifacts artifacts: 'target/dependency-check-report.html', followSymlinks: false
            }
        }
        
        stage('Sonarqube'){
            steps{
                figlet 'SonarQube'
                script{
                    def scannerHome = tool 'SonarQube Scanner'
                    
                    withSonarQubeEnv('Sonar Server'){
                        sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=ms-maven -Dsonar.sources=. -Dsonar.java.binaries=target/classes -Dsonar.exclusions='**/*/test/**/*, **/*/acceptance-test/**/*, **/*.html'"
                    }
                }
            }
        }


        // GENERACION DE RESPUESTA PARA SUCCESS O FAILED
        stage('Error') {
            // when doError is equal to 1, return an error
            when {
                expression { doError == '1' }
            }
            steps {
                figlet 'Slack Message Failure'
                echo "Failure :("
                error "Test failed on purpose, doError == str(1)"
            }
        }
        stage('Success') {
            // when doError is equal to 0, just print a simple message
            when {
                expression { doError == '0' }
            }
            steps {
                figlet 'Slack Message Success'
                echo "Success :)"
            }
        }
        stage('BuildUser') {
              steps {
                wrap([$class: 'BuildUser']) {
                  script {
                    varBUILD_USER = "${BUILD_USER}"
                    varBUILD_USER_ID = "${BUILD_USER_ID}"
                    varBUILD_USER_EMAIL = "${BUILD_USER_EMAIL}"
                  }
                  //sh 'echo "${BUILD_USER}"'
                  //sh 'echo "${BUILD_USER_ID}"'
                  //sh 'echo "${BUILD_USER_EMAIL}"'
                  echo varBUILD_USER
                  echo varBUILD_USER_ID
                  echo varBUILD_USER_EMAIL
                }
              }
         }
    }
        // Post-build actions
    post {
        always {
           script {
              BUILD_USER = varBUILD_USER
           }
            echo 'I will always say hello in the console.'
            slackSend channel: '#notificaction-jenkins',
                color: COLOR_MAP[currentBuild.currentResult],
                message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} by ${BUILD_USER}\n More info at: ${env.BUILD_URL}"
        }
    }      
}
