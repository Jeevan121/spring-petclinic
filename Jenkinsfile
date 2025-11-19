pipeline {
    agent any

    environment {
        DOCKERHUB = 'DOCKER_HUB_LOGIN'   // Credential ID in Jenkins
        APP_NAME = 'petclinic'
        DOCKER_IMAGE = "jeevan121/petclinic"
        MAVEN_HOME = tool 'maven3.9'
    }

    stages {
        
        stage('Checkout') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/Jeevan121/spring-petclinic.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh "${MAVEN_HOME}/bin/mvn clean package -DskipTests=true"
            }
        }

        stage('Unit Tests') {
            steps {
                sh "${MAVEN_HOME}/bin/mvn test"
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t $DOCKER_IMAGE:latest .
                '''
            }
        }

        stage('Login & Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKERHUB}", usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh '''
                    echo "$PASS" | docker login -u "$USER" --password-stdin
                    docker tag $DOCKER_IMAGE:latest $DOCKER_IMAGE:v1
                    docker push $DOCKER_IMAGE:v1
                    '''
                }
            }
        }

        stage('Deploy to EC2 via Ansible') {
            steps {
                sh '''
                ansible-playbook -i /tmp/inv deploy.yml
                '''
            }
        }
    }
}
