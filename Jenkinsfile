pipeline {
    agent any

    tools {
        jdk 'myjava'              // <-- USE JDK 25 FOR BUILD
        maven 'maven3.9'         // <-- matches your tool name
    }

    environment {
        // Explicit JAVA_HOME override for safety
        JAVA_HOME = "/usr/lib/jvm/temurin-25-jdk-amd64"
        PATH = "${JAVA_HOME}/bin:${PATH}"

        DOCKER_IMAGE = "jeevan11/petclinic"
        DOCKERHUB_CRED = "DOCKER_HUB_LOGIN"
    }

    stages {

        stage('Checkout') {
            steps {
                git url: 'https://github.com/Jeevan121/spring-petclinic.git', branch: 'main'
            }
        }

        stage('Verify Java & Maven Versions') {
            steps {
                sh 'java -version'
                sh 'mvn -version'
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
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
                withCredentials([usernamePassword(
                    credentialsId: "${DOCKERHUB_CRED}",
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
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
                    export ANSIBLE_HOST_KEY_CHECKING=False
                    ansible-playbook -i /tmp/inv deploy.yml
                '''
            }
        }
    }
}
