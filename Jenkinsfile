pipeline {
    agent any

    environment {
        GIT_REPO = 'https://github.com/basil1120/wso2-micro-integrator-apps.git'
        BRANCH = 'main'
        MAVEN_HOME = tool 'Maven'  // Ensure Maven is configured in Jenkins
        APIMCTL_PATH = '/Users/basam/Desktop/TMB/WSO2_SERVER/wso2-apictl/apictl-4.5.0'
        ENV_NAME = 'wso2-mi'  // WSO2 Micro Integrator environment name in APIMCTL
        MI_HOST = 'https://localhost:8253'
        MI_USERNAME = credentials('admin')  // Jenkins credential ID for MI username
        MI_PASSWORD = credentials('admin')  // Jenkins credential ID for MI password
    }

    stages {
        
        stage('Clone Repo') {
            git url: "${GIT_REPO}",
                credentialsId: 'CREDENTIALS_GITHUB',
                branch: "${BRANCH}"
        }

        stage('Build with Maven') {
            steps {
                sh "${MAVEN_HOME}/bin/mvn clean install"
            }
        }


        stage('Install APICTL') {
            steps {
                script {
                    sh """
                    if ! command -v apictl &> /dev/null; then
                        echo "Installing APICTL..."
                        curl -LO https://github.com/wso2/product-apim-tooling/releases/download/v4.3.0/apictl-4.3.0-linux-x64.tar.gz
                        tar -xzf apictl-4.3.0-linux-x64.tar.gz
                        sudo mv apictl /usr/local/bin/
                        chmod +x /usr/local/bin/apictl
                    else
                        apictl version
                        echo "APICTL already installed"
                    fi
                    """
                }
            }
        }

        stage('Clean & Remove APICTL Environments') {
            steps {
                script {
                    sh """
                    echo "---------- Listing APICTL Environments ---------"
                    apictl get envs | tail -n +2 | awk '{print \$1}' > apictl_envs.txt

                    echo "---------- Removing APICTL Environments ---------"
                    while read -r envName; do
                        if [ ! -z "\$envName" ]; then
                            echo "Removing APICTL environment: \$envName"
                            apictl remove env "\$envName"
                        fi
                    done < apictl_envs.txt
                    """
                }
            }
        }


        stage('Setup WSO2 MI Environment') {
            steps {
                script {
                    sh "${APIMCTL_PATH} mi add env ${ENV_NAME} --mi ${MI_HOST}"
                    sh "${APIMCTL_PATH} mi list envs"
                }
            }
        }

        stage('Deploy CAR File') {
            steps {
                script {
                    def carFile = sh(script: "find target -name '*.car'", returnStdout: true).trim()
                    if (carFile) {
                        sh "${APIMCTL_PATH} login ${ENV_NAME} -u ${MI_USERNAME} -p ${MI_PASSWORD} --verbose"
                        sh "${APIMCTL_PATH} mi import artifact --file ${carFile} -e ${ENV_NAME}"
                    } else {
                        error('CAR file not found!') //apictl mi import artefact --file test.car -e dev
                    } 
                }
            }
        }
    }

    post {
        success {
            echo 'Build and deployment successful!'
        }
        failure {
            echo 'Build or deployment failed!'
        }
    }
}