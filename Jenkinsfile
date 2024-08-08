pipeline {
    agent any 
    
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage ("Git Checkout") {
            steps {
                git branch: 'main', url: 'https://github.com/vijaygiduthuri/Netflix-Clone.git'
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage ("Build Docker Image") {
            steps {
                sh "docker build -t netflix ."
            }
        }
        stage ("Tag & Push to DockerHub") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker') {
                        sh "docker tag netflix vijaygiduthuri/netflix:latest"
                        sh "docker push vijaygiduthuri/netflix:latest"
                    }
                }
            }
        }
        stage ("Docker Scout Image Analysis ") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker') {
                        sh 'docker-scout quickview vijaygiduthuri/netflix:latest'
                        sh 'docker-scout cves vijaygiduthuri/netflix:latest'
                        sh 'docker-scout recommendations vijaygiduthuri/netflix:latest'
                    }
                }
            }
        }
        stage ("Deploy to Docker Conatiner") {
            steps {
                sh "docker run -itd --name netflix -p 4000:80 netflix:latest"
            }
        }
    }
    post {
    always {
        emailext attachLog: true,
            subject: "'${currentBuild.result}'",
            body: """
                <html>
                <body>
                    <div style="background-color: #FFA07A; padding: 10px; margin-bottom: 10px;">
                        <p style="color: white; font-weight: bold;">Project: ${env.JOB_NAME}</p>
                    </div>
                    <div style="background-color: #90EE90; padding: 10px; margin-bottom: 10px;">
                        <p style="color: white; font-weight: bold;">Build Number: ${env.BUILD_NUMBER}</p>
                    </div>
                    <div style="background-color: #87CEEB; padding: 10px; margin-bottom: 10px;">
                        <p style="color: white; font-weight: bold;">URL: ${env.BUILD_URL}</p>
                    </div>
                </body>
                </html>
            """,
            to: 'Provide_email_here',
            mimeType: 'text/html'
        }
    }
}    
