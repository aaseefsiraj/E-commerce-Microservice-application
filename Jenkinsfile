pipeline {
    agent any

    environment {
        AWS_REG = 'ap-northeast-1'
        // Add other env vars here if needed
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'master', url: 'https://github.com/Methx17/E-commerce-Microservice-application.git'
            }
        }

        stage('Configure & Build') {
            steps {
               withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'aws-cred', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) 
               {
                    sh """
                    # Setup Kubeconfig
                    aws eks update-kubeconfig --region ${AWS_REG} --name demo-cluster
                    
                    # Run your build script
                    chmod +x Microservices/docker_image_buid_push.sh
                    ./Microservices/docker_image_buid_push.sh
                    """
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                // IMPORTANT: kubectl needs the AWS keys to authenticate with the EKS cluster
                 withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'aws-cred', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) 
               {
                    sh "kubectl apply -f Microservices/kubernetes-manifests"
                }
            }
        }

        stage('Verify') {
            steps {
                 withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'aws-cred', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) 
               {
                    sh "kubectl get pods"
                }
            }
        }
    }
    // Optional - to delete the unwanted cache.
    post {
        always {
            // Cleanup to prevent the "No Space" error from coming back
            sh 'docker system prune -f'
            cleanWs()
        }
    }
}
