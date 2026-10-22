pipeline {
    agent any

    parameters {
        choice(name: 'IMAGE_TAG', choices: ['v1', 'v2-faulty'], description: 'Select image version to deploy')
    }

    environment {
        DOCKER_IMAGE = "nandana200621/agile-final-lab"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/nandanaj2006/Agile-final-lab.git'
            }
        }

        stage('Build & Test') {
            steps {
                bat '''
                echo "Validating web portal files..."
                python -c "import os; assert os.path.exists('index.html'); print('Portal verification successful.')"
                echo Portal Build Status: Verified for tag %IMAGE_TAG% > portal-report.txt
                '''
            }
        }

        stage('Archive Build Artifacts') {
            steps {
                archiveArtifacts artifacts: 'portal-report.txt', fingerprint: true
            }
        }

        stage('Build Docker Image') {
            steps {
                bat 'docker build -t %DOCKER_IMAGE%:%IMAGE_TAG% .'
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    bat 'docker login -u %USER% -p %PASS%'
                    bat 'docker push %DOCKER_IMAGE%:%IMAGE_TAG%'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kuberconfig', variable: 'KUBECONFIG')]) {
                    bat '''
                    set KUBECONFIG=%KUBECONFIG%
                    kubectl apply -f deployment.yaml --validate=false
                    kubectl set image deployment/disaster-portal disaster-portal=%DOCKER_IMAGE%:%IMAGE_TAG% --record
                    '''
                }
            }
        }
    }
}
