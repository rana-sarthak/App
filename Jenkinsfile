pipeline {
    agent any
    parameters {
        choice(name: 'ACTION', choices: ['Deploy', 'Destroy'], description: 'Select the action')
    }
    environment {
        AWS_CREDENTIALS = credentials('IAM User')
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'master', url: 'https://github.com/rana-sarthak/App.git'
                script{
                    bat '''
                        dir
                    '''
                }
            }
        }
        stage('Terraform Apply Plan') {
            when {
                expression {
                    params.ACTION == 'Deploy'
                }
            }
            steps {
                script {
                    bat '''
                        dir
	                    echo 'Running Terraform Apply Plan'
	                    move /Y main.tf "X:\\Code Workspace"
                        dir
	                    cd /d X:\\Code Workspace
	                    terraform plan -out=tfplan
                        dir
	                    echo 'Running Terraform Apply'
	                    terraform apply -auto-approve tfplan
                        dir
                    '''
                }
            }
        }
        stage('Login to AWS ECR') {
            when {
                expression {
                    params.ACTION == 'Deploy'
                }
            }
            steps {
                script {
                    bat '''
                        dir
	                    docker --version
                        aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 851725306571.dkr.ecr.us-east-1.amazonaws.com
                    '''
                }
            }
        }
        stage('Build Docker Image') {
            when {
                expression {
                    params.ACTION == 'Deploy'
                }
            }
            steps {
                script{
                    bat '''
                        docker build -t 851725306571.dkr.ecr.us-east-1.amazonaws.com/codeimage:latest .
                    '''
                }
            }
        }
        stage('Push Docker Image to ECR') {
            when {
                expression {
                    params.ACTION == 'Deploy'
                }
            }
            steps {
                script{
                    bat '''
                        docker push 851725306571.dkr.ecr.us-east-1.amazonaws.com/codeimage:latest
                    '''
                }
            }
        }
        stage('Terraform Destroy plan') {
            when {
                expression {
                    params.ACTION == 'Destroy'
                } 
            }
            steps {
                script {
                    echo 'Running Terraform Destroy plan'
                    bat '''
                        move /Y main.tf "X:\\Code Workspace"
	                    cd /d X:\\Code Workspace
	                    dir
                        terraform plan -destroy
                        dir
                        terraform destroy -auto-approve
                    '''
                }
            }
        }
    }
}
    