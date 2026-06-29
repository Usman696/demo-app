pipeline {
    agent any

    environment {
        REGISTRY = 'host.docker.internal:5001'
        IMAGE_NAME = 'demo-app'
        IMAGE_TAG = 'latest'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Tests') {
   	   steps {
               sh '''
            	   python3 -m venv venv
           	   . venv/bin/activate
            	   pip install --upgrade pip
           	   pip install -r requirements.txt
           	   pytest -v
               '''
   	   }
        }

        stage('Build image') {
            steps {
                sh '''
                    docker build -t $REGISTRY/$IMAGE_NAME:$IMAGE_TAG .
                '''
            }
        }
   
        stage('Docker login') {
            steps {
                sh '''
                    echo "mypassword" | docker login $REGISTRY -u jenkins --password-stdin
                '''
            }
        }

        stage('Push image') {
            steps {
                sh '''
                    docker push $REGISTRY/$IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    docker rm -f demo-app || true

                    docker run -d \
                      --name demo-app \
                      --restart unless-stopped \
                      -p 8000:5000 \
                      $REGISTRY/$IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }
    }
}
