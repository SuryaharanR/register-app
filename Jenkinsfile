pipeline {
    agent { label 'Jenkins-Agent' }

    tools {
        jdk 'Java17'
        maven 'Maven3'
    }

    environment {
        APP_NAME    = "register-app-pipeline"
        RELEASE     = "1.0.0"
        DOCKER_USER = "suryaharanr"
        IMAGE_NAME  = "suryaharanr/${APP_NAME}"
        IMAGE_TAG   = "${RELEASE}-${BUILD_NUMBER}"
    }

    stages {
        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }

        stage("Checkout from SCM") {
            steps {
                git branch: 'main',
                    credentialsId: 'github',
                    url: 'https://github.com/SuryaharanR/register-app'
            }
        }

        stage("Build Application") {
            steps {
                sh "mvn clean package"
            }
        }

        stage("Test Application") {
            steps {
                sh "mvn test"
            }
        }

        stage("SonarQube Analysis") {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') {
                        sh "mvn sonar:sonar"
                    }
                }
            }
        }

        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token'
                }
            }
        }

        stage("Docker Build & Push") {
            steps {
                script {
                    withDockerRegistry([credentialsId: 'dockerhub', url: 'https://index.docker.io/v1/']) {
                        def dockerImage = docker.build("suryaharanr/register-app:${BUILD_NUMBER}")
                        dockerImage.push("${BUILD_NUMBER}")
                        dockerImage.push("latest")
                    }
                }
            }
        
        }
        

        stage("Trivy Scan") {
            steps {
                script {
                    sh """
                    docker run -v /var/run/docker.sock:/var/run/docker.sock \
                    aquasec/trivy image suryaharanr/register-app:latest \
                    --no-progress --scanners vuln --exit-code 0 \
                    --severity HIGH,CRITICAL --format table
                    """
                }
            }
        }

        stage("Cleanup Artifacts") {
            steps {
                script {
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG} || true"
                    sh "docker rmi ${IMAGE_NAME}:latest || true"
                }
            }
        }

        // stage("Trigger CD Pipeline") {
        //     steps {
        //         script {
        //             sh """
        //             curl -v -k --user clouduser:${JENKINS_API_TOKEN} \
        //             -X POST -H 'cache-control: no-cache' \
        //             -H 'content-type: application/x-www-form-urlencoded' \
        //             --data 'IMAGE_TAG=${IMAGE_TAG}' \
        //             'http://ec2-13-232-128-192.ap-south-1.compute.amazonaws.com:8080/job/gitops-register-app-cd/buildWithParameters?token=gitops-token'
        //             """
        //         }
        //     }
        // }
    }

    // post {
    //     failure {
    //         emailext(
    //             body: '''${SCRIPT, template="groovy-html.template"}''',
    //             subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - Failed",
    //             mimeType: 'text/html',
    //             to: "ashfaque.s510@gmail.com"
    //         )
    //     }
    //     success {
    //         emailext(
    //             body: '''${SCRIPT, template="groovy-html.template"}''',
    //             subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - Successful",
    //             mimeType: 'text/html',
    //             to: "ashfaque.s510@gmail.com"
    //         )
    //     }
    // }
}
