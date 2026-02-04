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
        timeout(time: 10, unit: 'MINUTES')
        disableConcurrentBuilds()
    }

    stages {

        stage('Read Version') {
            steps {
                script {
                    def packageJSON = readJSON file: 'package.json'
                    appVersion = packageJSON.version
                    echo "app version: ${appVersion}"
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
            steps {
                withCredentials([string(credentialsId: 'GITHUB_TOKEN', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                        echo "🔍 Fetching Dependabot alerts..."

                        API_URL="https://api.github.com/repos/bandlamud/catalogue1/dependabot/alerts"

                        response=$(curl -s \
                          -H "Authorization: Bearer $GITHUB_TOKEN" \
                          -H "Accept: application/vnd.github+json" \
                          "$API_URL?per_page=100")

                        # Safety check for GitHub errors
                        if echo "$response" | jq -e '.message?' >/dev/null; then
                            echo "❌ GitHub API error:"
                            echo "$response"
                            exit 1
                        fi

                        high_critical_open_count=$(echo "$response" | jq '
                          [
                            .[] |
                            select(
                              .state=="open" and
                              (
                                .security_advisory.severity=="high" or
                                .security_advisory.severity=="critical"
                              )
                            )
                          ] | length
                        ')

                        echo "Open HIGH/CRITICAL alerts: $high_critical_open_count"

                        if [ "$high_critical_open_count" -gt 0 ]; then
                            echo "❌ Blocking pipeline due to OPEN HIGH/CRITICAL alerts"
                            echo "Affected dependencies:"
                            echo "$response" | jq -r '
                              .[] |
                              select(
                                .state=="open" and
                                (
                                  .security_advisory.severity=="high" or
                                  .security_advisory.severity=="critical"
                                )
                              ) |
                              "- \(.dependency.package.name) | \(.security_advisory.severity) | \(.security_advisory.summary)"
                            '
                            exit 1
                        else
                            echo "✅ No OPEN HIGH/CRITICAL Dependabot alerts found"
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
