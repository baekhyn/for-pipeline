pipeline{
    agent any
    
    stages{
        stage('Build Image'){
            steps{
                script{
                    dockerImage = docker.build(IMAGE_NAME)
                }
            }
        }
        
        stage('Push Image'){
            steps{
                script{
                    sh "echo ${BUILD_NUMBER}"
                    docker.withRegistry("https://${ECR_PATH}", "ecr:${REGION}:${AWS_CREDENTIAL_ID}"){
                        dockerImage.push("v"+ BUILD_NUMBER)
                    }
                    sh "docker rmi ${ECR_PATH}:v${BUILD_NUMBER}"
                }
            }
        }

        stage('Deploy') {
            steps{
                git credentialsId: "${GITHUB_CREDENTIAL_ID}",
                    url: 'https://github.com/baekhyn/for-manifest.git',
                    branch: 'main'
                sh(
                    """
                    #!/usr/bin/env bash
                    sed -i -E 's/(image: .+):[a-zA-Z0-9_-]+/\\1:v${BUILD_NUMBER}/' ${EKS_YAML} ${OC_YAML}
                    cat ${EKS_YAML} ${OC_YAML}
                    git add ${EKS_YAML} ${OC_YAML}
                    git commit -m "update the image tag:${BUILD_NUMBER}"
                    """
                )
                sshagent(credentials: ["${SSH_CREDENTIAL}"]){
                    sh(
                        """
                        #!/usr/bin/env bash
                        export GIT_SSH_COMMAND="ssh -oStrictHostKeyChecking=no"
                        git remote set-url origin git@github.com:baekhyn/for-manifest.git
                        git push -u origin main
                        """
                    )
                }
            }
        }
    }

    post {
        success{
            slackSend(
                channel: '#backend #devops',
                color: '#0ac914',
                message: 
                """
                ✅ SUCCEEDED\n ✏️ Job: ${JOB_NAME} - [#${BUILD_NUMBER}]\n 🔗 URL: ${BUILD_URL}
                """
            )
        }
        failure{
            slackSend(
                channel: '#backend #devops',
                color: '#c90a30',
                message: 
                """
                ⚠️ FAILED\n ✏️ Job: ${JOB_NAME} - [#${BUILD_NUMBER}]\n 🔗 URL: ${BUILD_URL}
                """
            )
        }
    }
}