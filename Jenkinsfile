pipeline {
    agent any
    tools { maven 'Maven3' }   // configure in Manage Jenkins → Tools

    stages {
        stage('Checkout') {
            steps { checkout scm }
        }
        stage('Build') {
            steps { sh 'mvn clean package -DskipTests' }
        }
        stage('Test') {
            steps { sh 'mvn test' }
            post {
                always { junit 'target/surefire-reports/*.xml' }
            }
        }
        stage('Deploy to EC2') {
            when { branch 'main' }
            steps {
                sshagent(['ec2-ssh-key']) {
                    sh '''
                    scp -o StrictHostKeyChecking=no target/*.jar ec2-user@<EC2_PUBLIC_IP>:/home/ec2-user/app.jar
                    ssh -o StrictHostKeyChecking=no ec2-user@<EC2_PUBLIC_IP> "pkill -f app.jar || true; nohup java -jar /home/ec2-user/app.jar > app.log 2>&1 &"
                    '''
                }
            }
        }
    }
    post {
        success { echo 'Pipeline succeeded' }
        failure { echo 'Pipeline failed' }
    }
}
