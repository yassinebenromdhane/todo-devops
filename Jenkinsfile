pipeline {
    agent any

    environment {
        NODEJS_HOME = tool name: 'NodeJS 16', type: 'NodeJS'
        PATH = "${env.NODEJS_HOME}/bin:${env.PATH}"
        DOCKER_CREDENTIALS_ID = '586327ad-80a7-47a8-8f35-a594ca36bc5e'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/yassinebenromdhane/todo-devops'
            }
        }
        
        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }
        
        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'owasp-dp-check' , dependencyCheckPublisher pattern : '**/dependency-check-report.xml'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("yassine0707/todo-devops:${env.BUILD_ID}")
                }
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', {$env.DOCKER_CREDENTIALS_ID}) {
                        docker.image("josemokeni/expense-tracker:${env.BUILD_NUMBER}").push()
                        docker.image("josemokeni/expense-tracker:${env.BUILD_NUMBER}").push("latest")
                    }
                }
            }
        }
        
        stage('Deploy to Minikube') {
            steps {
                script {
                    withKubeConfig([credentialsId: 'minikube-kubeconfig']) {
                        sh 'kubectl apply -f k8s/Deployment.yaml'
                    }
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
    }
}

