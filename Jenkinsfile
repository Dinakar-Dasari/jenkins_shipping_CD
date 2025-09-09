    pipeline{
        agent {
            label "agent-1"
        }

        tools {
            nodejs 'nodejs-20'
        }
        environment {   
            PROJECT = 'roboshop'
            COMPONENT = 'cart'
            REGION = 'us-east-1'
            ACC_ID = '127218179061'
        }

        parameters{
            string(name:'appVersion', description: "App version of the image" )
            choice(name:'deploy_to', choices: ["DEV", "QA", "PROD"], description: "Deploy Environment")
        }

        stages{
            stage('DEPLOY'){
                steps{
                    withAWS(credentials:'aws-creds', region: 'us-east-1'){
                        sh """
                            aws eks update-kubeconfig --region $REGION --name "$PROJECT-${params.deploy_to}"
                            kubectl apply -f 01-namespace.yaml
                            sed -i "s/IMAGE_VERSION/${params.appVersion}/g" values-${params.deploy_to}.yaml
                            helm upgrade --install $COMPONENT -f values-${params.deploy_to}.yaml -n $PROJECT .
                        """               
                    }
                }
            }

            stage('Deployment status') {
                steps {
                    script{
                        withAWS(credentials:'aws-creds', region: 'us-east-1'){
                            def deployment_status = sh(returnStdout: true, script:"kubectl rollout status deployment/${COMPONENT} --timeout=30s -n ${PROJECT}").trim()
                            if (deployment_status.contains("successfully rolled out")) {
                                echo "Deployment success"
                            } else {
                                sh "helm rollback ${COMPONENT} -n ${PROJECT}"
                                def rollback_status = sh(returnStdout: true, script:"kubectl rollout status deployment/${COMPONENT} --timeout=30s -n ${PROJECT}").trim()
                                if (rollback_status.contains("successfully rolled out")) {
                                error "Deployment failed, Rollback success"
                                } else {
                                    error "Deployment failed, Rollback failed. Application is not running !!"
                                }
                            }

                        }

                    }
                }
            }
        }

    }
