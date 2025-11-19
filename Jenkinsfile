pipeline {
    agent any

    environment {
        MAVEN_HOME = tool 'maven3.9'
        DOCKER_IMAGE = "jeevan121/petclinic"
        DOCKERHUB_CRED = "DOCKER_HUB_LOGIN"
    }

    stages {

        stage('Checkout') {
            steps {
                git url: 'https://github.com/Jeevan121/spring-petclinic.git', branch: 'master'
            }
        }

        stage('Maven Build') {
            steps {
                sh "${MAVEN_HOME}/bin/mvn clean package -DskipTests"
            }
        }

        stage('Run Tests') {
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

        stage('Docker Login & Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CRED}", usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh '''
                        echo "$PASS" | docker login -u "$USER" --password-stdin
                        docker tag $DOCKER_IMAGE:latest $DOCKER_IMAGE:v1
                        docker push $DOCKER_IMAGE:v1
                    '''
                }
            }
        }

        stage('Deploy via Ansible') {
            steps {
                sh '''
                    ansible-playbook -i /tmp/inv deploy.yml
                '''
            }
        }
    }
}
