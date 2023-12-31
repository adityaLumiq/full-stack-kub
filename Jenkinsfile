def api_url = "foo"
pipeline {
    agent any



    stages {
        stage('Clone') {
            steps {
                cleanWs()
                // Get some code from a GitHub repository
                sh 'git clone https://github.com/adityaLumiq/full-stack-kub.git'
                sh 'git clone https://github.com/dhruvrkashyap/docker-frontend-backend-db.git'
            }
        }
        stage('Build') {
            steps {
                sh 'cd docker-frontend-backend-db/backend && docker build -t api .'
                sh 'cd docker-frontend-backend-db/frontend && docker build -t web .'
            }
        }
        stage('Push') {
            steps {
                sh 'aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws/d3h2f0h9'
                sh 'docker tag web:latest public.ecr.aws/d3h2f0h9/web:latest'
                sh 'docker push public.ecr.aws/d3h2f0h9/web:latest'

                sh 'docker tag api:latest public.ecr.aws/d3h2f0h9/api:latest'
                sh 'docker push public.ecr.aws/d3h2f0h9/api:latest'
            }
        }
        stage ('Deploy '){
            steps{
                echo "My variable is  ${api_url}"
                catchError(buildResult: 'SUCCESS', stageResult: 'SUCCESS') {
                    sh "kubectl delete ns jenkins"
                }
                sh 'kubectl create ns jenkins'
                sh 'kubectl apply -f full-stack-kub/pvc.yaml -n jenkins'
                sh 'kubectl apply -f full-stack-kub/config.yaml -n jenkins'

                sh 'kubectl apply -f full-stack-kub/mongo-headless.yaml -f full-stack-kub/api-service.yaml -f full-stack-kub/web-service.yaml -n jenkins'
                
                sh "timeout 20s bash -c 'until kubectl -n jenkins get service/api --output=jsonpath='{.status.loadBalancer}' | grep ingress; do : ; done'"
                
                script{
                    api_url = sh (script: "kubectl -n jenkins get service/api --output=jsonpath='{.status.loadBalancer.ingress[0].hostname}'", returnStdout: true)
                    api_url = "http://${api_url}:3001/api"
                }
                echo "My variable is ${api_url}"
                
                sh "kubectl -n jenkins get cm config -o yaml | sed -e \"s|REACT_APP_API_URL:.*|REACT_APP_API_URL: ${api_url}| \" | kubectl apply -f -"

                sh 'kubectl apply -f full-stack-kub/mongo-ss.yaml -n jenkins'
                sh 'kubectl apply -f full-stack-kub/api-deployment.yaml -n jenkins'
                sh 'kubectl apply -f full-stack-kub/web-deployment.yaml -n jenkins'
            }
        }
        
    }
}
