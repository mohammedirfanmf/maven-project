pipeline {
    agent any

    tools {
        maven 'Maven'
        jdk 'JDK17'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/bhuvan-raj/MavenApp-deployment-to-Ec2-with-JenkinsPipeline.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(['ec2-ssh-key']) {
                    sh '''
                        echo "Copying artifact to EC2..."
                        scp -o StrictHostKeyChecking=no \
                            target/demo-1.0.0.jar \
                            ubuntu@<13.234.240.3>:/opt/app/

                        echo "Starting application on EC2..."
                        ssh -T -o StrictHostKeyChecking=no ubuntu@<13.234.240.3> '
                            pkill -f demo-1.0.0.jar || true
                            setsid nohup java -jar /opt/app/demo-1.0.0.jar \
                                > /opt/app/app.log 2>&1 < /dev/null &
                            exit 0
                        '
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "Deployment completed successfully."
        }
        failure {
            echo "Deployment failed. Check Jenkins or EC2 logs."
        }
    }
}
