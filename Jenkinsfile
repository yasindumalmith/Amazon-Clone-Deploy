pipeline {
    agent any

    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage("Clean Workspace") {
            steps {
                cleanWs()
            }
        }

        stage("Git Checkout") {
            steps {
                git branch: 'main', url: 'https://github.com/yasindumalmith/Amazon-Clone-Deploy.git'
            }
        }

        stage("SonarQube Analysis") {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=amazon \
                        -Dsonar.projectKey=amazon '''
                }
            }
        }

        stage("Quality Gate") {
            steps {
                script {
                    timeout(time: 3, unit: 'MINUTES') {
                        waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                    }
                }
            }
        }

        stage("Install NPM Dependencies") {
            steps {
                sh "npm install"
            }
        }
        
        stage("OWASP FS Scan") {
            steps {
                dependencyCheck additionalArguments: '''
                    --scan ./ 
                    --disableYarnAudit 
                    --disableNodeAudit 
                ''', odcInstallation: 'dp-check'

                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage("Trivy File Scan") {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }

        stage("Build Docker Image") {
            steps {
                script {
                    env.IMAGE_TAG = "yasindumalmith/amazon:${BUILD_NUMBER}"
                    // Optional cleanup
                    sh "docker build -t ${env.IMAGE_TAG} ."
                }
            }
        }

        stage("Tag & Push to DockerHub") {
            steps {
                script {
                    withCredentials([string(credentialsId: 'docker-cred', variable: 'dockerpwd')]) {
                        sh "docker login -u yasindumalmith -p ${dockerpwd}"
                        sh "docker push ${env.IMAGE_TAG}"
                    }
                }
            }
        }

        stage("Trivy Scan Image") {
            steps {
                script {
                    sh """
                    echo '🔍 Running Trivy scan on ${env.IMAGE_TAG}'

                    # JSON report
                    trivy image -f json -o trivy-image.json ${env.IMAGE_TAG}

                    # HTML report using built-in HTML format (saving as txt per your script)
                    trivy image -f table -o trivy-image.txt ${env.IMAGE_TAG}

                    # Fail build if HIGH/CRITICAL vulnerabilities found
                    # trivy image --exit-code 1 --severity HIGH,CRITICAL ${env.IMAGE_TAG} || true
                    """
                }
            }
        }

        stage("Update Helm Image Tag (GitOps)") {
            steps {
                script {
                    withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                        sh """
                        rm -rf amazon
                        git clone https://${GITHUB_TOKEN}@github.com/yasindumalmith/amazon-k8s-deploy.git 
                        cd amazon

                        # Update image tag in values.yaml
                        sed -i 's/tag:.*/tag: "${BUILD_NUMBER}"/' amazon/values.yaml

                        git config user.email "jenkins@ci.com"
                        git config user.name "jenkins"

                        git add amazon/values.yaml
                        git commit -m "Update amazon image to ${BUILD_NUMBER}"
                        git push origin main
                        """
                    }
                }
            }
        }
    } // End of stages block

    post {
        always {
            script {
                def buildStatus = currentBuild.currentResult
                def buildUser = currentBuild.getBuildCauses('hudson.model.Cause$UserIdCause')[0]?.userId ?: 'Github User'

                emailext (
                    subject: "Pipeline ${buildStatus}: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                    body: """
                        <p>This is a Jenkins Amazon CICD pipeline status.</p>
                        <p>Project: ${env.JOB_NAME}</p>
                        <p>Build Number: ${env.BUILD_NUMBER}</p>
                        <p>Build Status: ${buildStatus}</p>
                        <p>Started by: ${buildUser}</p>
                        <p>Build URL: <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
                    """,
                    to: 'wwwyasindumalmith7@gmail.com',
                    from: 'wwwyasindumalmith7@gmail.com',
                    mimeType: 'text/html',
                    attachmentsPattern: 'trivyfs.txt,trivy-image.json,trivy-image.txt,dependency-check-report.xml'
                )
            }
        }
    } // End of post block
} // End of pipeline block