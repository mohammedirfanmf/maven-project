pipeline {
    agent any

    tools {
        maven 'Maven'
        jdk 'JDK17'
    }

    environment {
        APP_NAME = "jenkins-demo"
        VERSION = "1.0.0"
        EC2_USER = "ubuntu"
        EC2_HOST = "13.234.240.3"
        DEPLOY_DIR = "/opt/app"
        JAR_NAME = "${APP_NAME}-${VERSION}.jar"
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
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(['ec2-ssh-key']) {
                    sh '''
                        echo "Creating app directory..."
                        ssh -o StrictHostKeyChecking=no $EC2_USER@$EC2_HOST \
                        "mkdir -p $DEPLOY_DIR"

                        echo "Copying JAR to EC2..."
                        scp -o StrictHostKeyChecking=no \
                        target/$JAR_NAME \
                        $EC2_USER@$EC2_HOST:$DEPLOY_DIR/

                        echo "Stopping old app (if running)..."
                        ssh -o StrictHostKeyChecking=no $EC2_USER@$EC2_HOST \
                        "pkill -f $JAR_NAME || true"

                        echo "Starting new app..."
                        ssh -o StrictHostKeyChecking=no $EC2_USER@$EC2_HOST "
                            nohup java -jar $DEPLOY_DIR/$JAR_NAME \
                            > $DEPLOY_DIR/app.log 2>&1 &
                        "
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ Deployment completed successfully!"
        }
        failure {
            echo "❌ Deployment failed. Check logs."
        }
    }
}
