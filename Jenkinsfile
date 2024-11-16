pipeline {
    agent {
        label 'master'
    }

    environment {
        HARBOR_URL = "harbor.fn-k8s.fishnet.com:32022" // 替换为你的 Harbor 地址
        HARBOR_PROJECT = "fishnet"      // 替换为你的 Harbor 项目名称
        DOCKER_IMAGE = "${HARBOR_URL}/${HARBOR_PROJECT}/my-test"
        K8S_NAMESPACE = "fishnet-dev"         // Kubernetes 命名空间
        DEPLOYMENT_FILE = "deployment.yaml" // Kubernetes 部署名称
    }

    stages {
        stage('Build Application') {
            steps {
                sh './mvnw clean package -DskipTests'
            }
        }

//         stage('Build Docker Image') {
//             steps {
//                 sh '''
//                 docker build -t ${DOCKER_IMAGE}:latest .
//                 '''
//             }
//         }

        stage('Push Docker Image to Harbor') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'harbor-credentials', usernameVariable: 'HARBOR_USERNAME', passwordVariable: 'HARBOR_PASSWORD')]) {
                    sh '''
                    echo "$HARBOR_PASSWORD" | docker login ${HARBOR_URL} -u "$HARBOR_USERNAME" --password-stdin
                    docker push ${DOCKER_IMAGE}:latest
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                kubectl apply -f template/${DEPLOYMENT_FILE} -n ${K8S_NAMESPACE}
                '''
            }
        }
    }

    post {
        success {
            echo 'Deployment to Kubernetes was successful!'
        }
        failure {
            echo 'Deployment to Kubernetes failed!'
        }
    }
}
