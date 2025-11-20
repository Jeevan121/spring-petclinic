pipeline {
    agent any

    tools {
        jdk 'myjava'                 // Your JDK 25 tool name
        maven 'maven3.9'             // Your Maven tool name
    }

    environment {
        JAVA_HOME = "/usr/lib/jvm/temurin-25-jdk-amd64"
        PATH = "${JAVA_HOME}/bin:${PATH}"

        DOCKER_IMAGE = "jeevan11/petclinic"      // Updated Docker repo
        DOCKERHUB_CRED = "DOCKER_HUB_LOGIN"      // DockerHub credential ID
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
                script {
                    env.BUILD_VERSION = "v${env.BUILD_NUMBER}"

                    sh """
                        docker build -t ${DOCKER_IMAGE}:latest .
                        docker tag ${DOCKER_IMAGE}:latest ${DOCKER_IMAGE}:${BUILD_VERSION}
                    """
                }
            }
        }

        stage('Docker Login & Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: DOCKERHUB_CRED,
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {

                    sh '''
                        echo "$PASS" | docker login -u "$USER" --password-stdin
                    '''
                }

                sh """
                    docker push ${DOCKER_IMAGE}:latest
                    docker push ${DOCKER_IMAGE}:${BUILD_VERSION}
                """
            }
        }

        stage('Deploy via Ansible') {
            steps {
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'ansible-ssh-key',
                    usernameVariable: 'SSH_USER',
                    keyFileVariable: 'SSH_KEY'
                )]) {

                    sh '''
                        export ANSIBLE_HOST_KEY_CHECKING=False
                        ansible-playbook -i /tmp/inv deploy.yml \
                            --private-key $SSH_KEY \
                            -u $SSH_USER
                    '''
                }
            }
        }

    } // end stages
} // end pipeline
