pipeline {
    agent any
    environment {
        AWS_REGION= 'ap-south-1'
        ECR_REPO= '607709788195.dkr.ecr.ap-south-1.amazonaws.com/maven-web-app'
        IMAGE_TAG= "${BUILD_NUMBER}"
     }
    
    tools{
        maven 'maven3.9.12'
    }

    stages {
        stage('CheckOutCode') {
            steps {
                git 'https://github.com/mirzatechno/maven-web-application.git'
            }
        }
        stage('build-package') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Code-analysis') {
            steps {
                sh 'mvn sonar:sonar'
            }
        }
        stage('buidImage') {
            steps {
                sh 'docker build -t $ECR_REPO:$IMAGE_TAG .'
            }   
        }
        stage('ECRlogin') {
            steps {
                sh '''
                aws ecr get-login-password --region $AWS_REGION \
                | docker login --username AWS --password-stdin $ECR_REPO
                '''
            }   
        }
        stage('Docker-Push') {
            steps {
                sh 'docker push $ECR_REPO:$IMAGE_TAG'
            }   
        }
        stage('Deploy to EKS') {
            steps {
                sh '''
                aws eks update-kubeconfig \
                --region $AWS_REGION \
                --name prod-eks

                # Replace image in deployment
                sed -i "s|IMAGE_PLACEHOLDER|$ECR_REPO:$IMAGE_TAG|g" MavenWebApplication.yaml

                # Apply resources in order
                kubectl apply -f MavenWebApplication.yaml
               
                kubectl apply -f ingress.yaml
                kubectl apply -f HPA.yaml

                # Verify rollout
                kubectl rollout status deployment/webpage-deployment -n production
                '''
            }
        }

        
    }// stages close
}//pipeline close
