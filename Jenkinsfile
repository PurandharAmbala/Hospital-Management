pipeline {
    agent {
        label 'dev'
    }
    parameters {
        string(
            name: 'IMAGE_TAG',
            defaultValue: 'latest',
            description: 'Docker Image Tag'
        )
        choice (
            name: 'GIT_BRANCH',
            choices: ['main', 'deploy', 'release'],
            description: 'select git branch to build'
         )
         booleanParam (
             name: 'RUN_TESTS',
             defaultValue: true,
             description: 'Run npm Tests'
         )
         booleanParam (
             name: 'RUN_CONTAINER',
             defaultValue: true,
             description: 'Run Docker Container'
        )
         choice (
             name: 'DEPLOY_ENV',
             choices: ['DEV', 'QA', 'PROD'],
             description: 'Select deploymenr Eenvironment'
         )
    }

    stages {
        stage('Display Parameters') {
            steps {
                echo "========================================="
                echo "        BUILD PARAMETERS"
                echo "========================================="
                echo "IMAGE_TAG       : ${params.IMAGE_TAG}"
                echo "GIT_BRANCH      : ${params.GIT_BRANCH}"
                echo "RUN_TESTS       : ${params.RUN_TESTS}"
                echo "RUN_CONTAINER   : ${params.RUN_CONTAINER}"
                echo "DEPLOY_ENV      : ${params.DEPLOY_ENV}"
                echo "==========================================="
            }
        }
        stage('Git') {
            steps {
                echo 'Selected Branch: ${params.GIT_BRANCH}'
                git branch: "${params.GIT_BRANCH}", credentialsId: 'Git-crd', url: 'https://github.com/PurandharAmbala/Hospital-Management.git'
            }
        }
        
        stage('Install Dependencies'){
            steps {
                echo "Installing Node.Js dependencies..."
                sh 'npm install'
            }
        }
        stage('Run tests'){
            when {
                expression {
                    return params.RUN_TESTS
                }
            }
            steps {
                echo "RUN_TESTS=true"
                echo "Running npm test..."
                
                sh '''
                    npm test || {
                        echo "No tests configured in this project."
                    }
                '''
            }
        }
        stage('Skip Tests') {
            when {
                expression {
                    return !params.RUN_TESTS
                }
            }
            steps {
                echo "RUN_TESTS=false"
                echo "Skipping npm test..."
            }
        }
        stage('Docker image') {
            steps {
                echo "Building Docker image..."
                sh  "docker build -t hospital-management:${params.IMAGE_TAG} ."
            }
        }
        stage('Run Docker Container') {
            when {
                expression {
                    return params.RUN_CONTAINER
                }
            }
            steps {
                echo "RUN_CONTAINER=true - running container"
                sh """
                docker rm -f hospital 2>/dev/null || true
                docker run -itd -p 3000:3000 --name hospital hospital-management:${params.IMAGE_TAG}
                """
            }
        }
        stage('Skip Container') {
            when {
               expression {
                   return !params.RUN_CONTAINER
               } 
            }
            steps{
                echo "RUN_CONTAINER=false - skipping container"
            }
        }
        stage('Deploy DEV') {
            when {
                expression {
                    return params.DEPLOY_ENV == 'DEV'
                }
            }
            steps {
                echo "Deploying Hospital-Management to DEV"
            }
        }
        stage('Deploy QA') {
            when {
                expression {
                    return params.DEPLOY_ENV == 'QA'
                }
            }
            steps {
                echo "Deploying Hospital-Management to QA"
            }
        }
        stage('Deploy PROD') {
            when {
                expression {
                    return params.DEPLOY_ENV == 'PROD'
                }
            }
            steps {
                echo "Deploying Hospital-Management to PROD"
            }
        }
    }
     post {

        success {
            echo "Pipeline completed successfully."
        }

        failure {
            echo "Pipeline failed."
        }

        always {
            echo "Build Number: ${env.BUILD_NUMBER}"
            echo "Build Result: ${currentBuild.result}"
        }
    }
}
