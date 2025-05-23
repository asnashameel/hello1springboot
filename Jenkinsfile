pipeline {
    agent any
    environment {
        JAVA_IMAGE='asnashameel/java-app:latest'
    }

    stages {
        stage('checkout') {
            steps {
                git url:'https://github.com/asnashameel/hello1springboot', branch:'main'
            }
        }
        stage('docker build') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'mytoken', usernameVariable: 'USER_NAME', passwordVariable: 'PASSWD')]){
                sh '''
                docker build -t $JAVA_IMAGE .
                
                if docker ps -a | grep java-app-container; then
                   docker stop 'java-app-container'
                   docker rm 'java-app-container'
                fi
                
                docker login -u $USER_NAME -p $PASSWD
                docker push $JAVA_IMAGE
                docker run -d -p 8001:8000 --name java-app-container $JAVA_IMAGE
                '''
                }
            }
        }
        stage('kubernetes') {
            steps {
                sh '''
                kubectl delete deploy --all
                kubectl create deployment my-deploy --image=$JAVA_IMAGE --dry-run=client -o yaml > deploy.yaml
                kubectl apply -f deploy.yaml
                kubectl delete svc --all
                kubectl expose deploy my-deploy --name my-svc --type=ClusterIP --port=8001 --target-port=8000
                '''
            }
        }
    }
}
