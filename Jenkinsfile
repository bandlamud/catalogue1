pipeline {
    agent {
        node {
            label 'AGENT-1'
        }
    }

    environment {
        COURSE = "Jenkins"
        appVersion = ""
        ACC_ID = "022779559954"
        PROJECT = "roboshop"
        COMPONENT = "catalogue"
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

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Unit Test') {
            steps {
                sh 'npm test'
            }
        }

        stage('Dependabot Security Gate') {
            environment {
                GITHUB_OWNER = 'bandlamud'
                GITHUB_REPO  = 'catalogue1'
                GITHUB_API   = 'https://api.github.com/repos/bandlamud/catalogue1/dependabot/alerts'
            }

            steps {
                withCredentials([string(credentialsId: 'GITHUB_TOKEN', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                        echo "🔍 Fetching Dependabot alerts..."

                        ALERTS_JSON=$(curl -s \
                          -H "Accept: application/vnd.github+json" \
                          -H "Authorization: Bearer $GITHUB_TOKEN" \
                          -H "X-GitHub-Api-Version: 2022-11-28" \
                          $GITHUB_API/repos/$GITHUB_OWNER/$GITHUB_REPO/dependabot/alerts)

                        echo "🚦 Evaluating OPEN + HIGH / CRITICAL alerts..."

                        FAIL_COUNT=$(echo "$ALERTS_JSON" | jq '
                          [
                            .[] |
                            select(
                              .state == "open" and
                              (
                                .security_advisory.severity == "high" or
                                .security_advisory.severity == "critical"
                              )
                            )
                          ] | length
                        ')

                        if [ "$FAIL_COUNT" -gt 0 ]; then
                            echo "❌ BLOCKING PIPELINE: $FAIL_COUNT OPEN HIGH/CRITICAL Dependabot alerts found"
                            exit 1
                        else
                            echo "✅ PASS: No OPEN HIGH/CRITICAL Dependabot alerts found"
                        fi
                    '''
                }
            }
        }

        stage('Build Image') {
            steps {
                withAWS(region: 'us-east-1', credentials: 'aws-creds') {
                    sh """
                        aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com
                        docker build -t ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion} .
                        docker push ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion}
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
