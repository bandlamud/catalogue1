pipeline {
    // this is the pre-build section
    agent {
        node {
            label 'AGENT-1'
                
        }
    }
    environment {
        COURSE = "Jenkins"
        appVersion = " "
    }
    options {
                timeout(time: 1, unit: 'HOURS') 
                disableConcurrentBuilds()
            }
    stages {
        stage('Read Version') {
            steps {
                script {
                    def packageJSON = readJSON file: 'package.json'
                    appVersion = packageJSON.version
                    echo "package version = ${appVersion}"
                }
               
            }
        }
        stage('Test') {
            steps {
               script {

                    sh """
                        echo "Testing"
                    """
               }
            }
        }
        stage('Deploy') {
            //     input {
            //     message "Should we continue?"
            //     ok "Yes, we should."
            //     submitter "alice,bob"
            //     parameters {
            //         string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')
            //     }
            // }
            when {
                expression { "$params.DEPLOY" == "true" }
            }
            steps {
                script {

                    sh """
                        echo "Deploying"
                    """
                }
            }
        }
    }
    post { 
        always { 
            echo 'I will always say Hello again!'
            cleanWs()
            }
            success {
              echo 'I will run if success'  
            }
            failure {
              echo 'I will run if failure'  
            }
            aborted {
                echo 'Pipeline is aborted'
            }
        }
     }
        