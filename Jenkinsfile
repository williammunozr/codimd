pipeline { 

    environment { 

        // Variables

        REGION_ID = "us-east-2"
        EKS_CLUSTER_NAME = "eks-cluster"
        REGISTRY = "hachikoapp/codimd" 
        dockerImage = '' 

        GIT_COMMIT_SHORT = sh(
                script: "printf \$(git rev-parse --short ${GIT_COMMIT})",
                returnStdout: true
        )

        // Getting Secrets

        ACCOUNT_ID = credentials('AWS_ACCOUNT_ID')
        DB_NAME = credentials('DB_NAME')
        DB_ENDPOINT = credentials('DB_ENDPOINT')
        DB_USERNAME = credentials('DB_USERNAME')
        DB_PASSWORD = credentials('DB_PASSWD')
        DB_DIALECT = credentials('DB_DIALECT')
        
        registryCredential = 'dockerHubId' 

    }

    agent any 

    stages { 

        stage('Building Codimd Image') { 
            steps { 
                script { 
                    sh 'pwd'
                    dockerImage = docker.build("${REGISTRY}:${env.GIT_COMMIT_SHORT}", "${WORKSPACE}/deployments")
                }
            } 
        }

        stage('Deploy Codimd Image') { 
            steps { 
                script { 
                    docker.withRegistry( '', registryCredential ) { 
                        dockerImage.push() 
                    }
                } 
            }
        } 

        stage('Cleaning Up') { 
            steps { 
                sh "docker rmi $REGISTRY:$GIT_COMMIT_SHORT" 
            }
        }

        stage('CodiMD Deployment to EKS') {
            steps {
                script {
                  sh """
                    aws eks --region $REGION_ID update-kubeconfig --name $EKS_CLUSTER_NAME
                    sed -i 's/{{ACCOUNT_ID}}/$ACCOUNT_ID/g' deployments/codimd-service.yaml
                    kubectl apply -f deployments/codimd-service.yaml

                    sed -i 's/{{TAG}}/$GIT_COMMIT_SHORT/g' deployments/codimd-deployment.yaml
                    sed -i 's/{{DB_NAME}}/$DB_NAME/g' deployments/codimd-deployment.yaml
                    sed -i 's/{{DB_USERNAME}}/$DB_USERNAME/g' deployments/codimd-deployment.yaml
                    sed -i 's/{{DB_PASSWORD}}/$DB_PASSWORD/g' deployments/codimd-deployment.yaml
                    sed -i 's/{{DB_DIALECT}}/$DB_DIALECT/g' deployments/codimd-deployment.yaml
                    sed -i 's/{{DB_ENDPOINT}}/$DB_ENDPOINT/g' deployments/codimd-deployment.yaml
                    kubectl apply -f deployments/codimd-deployment.yaml
                  """
                }
            }
        }
    }
}