def api_url = "foo"
pipeline {
    agent any



    stages {
        stage('Build') {
            steps {
                cleanWs()
                // Get some code from a GitHub repository
                sh 'git clone https://github.com/adityaLumiq/full-stack-kub.git'
            }
        }
        stage ('kub'){
            steps{
                echo "My variable is  ${api_url}"
                sh 'kubectl create ns jenkins'
                sh 'kubectl apply -f full-stack-kub/api-deployment.yaml -n jenkins'
                sh 'kubectl apply -f full-stack-kub/api-service.yaml -n jenkins'
                sh "timeout 20s bash -c 'until kubectl -n jenkins get service/api --output=jsonpath='{.status.loadBalancer}' | grep ingress; do : ; done'"
                script{
                    api_url = sh (script: "kubectl -n jenkins get service/api --output=jsonpath='{.status.loadBalancer.ingress[0].hostname}'", returnStdout: true)
                }
                echo "My variable is ${api_url}"
                sh "kubectl -n jenkins get cm config -o yaml | sed -e \"s|REACT_APP_API_URL:.*|REACT_APP_API_URL: ${api_url}| \" | kubectl apply -f -"
    
                sh 'kubectl apply -f full-stack-kub/web-deployment.yaml -n jenkins'
                sh 'kubectl apply -f full-stack-kub/web-service.yaml -n jenkins'
            }
        }
        
    }
}
