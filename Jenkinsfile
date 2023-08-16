pipeline {
    
    environment {
        // Set your AWS credentials environment variables
        AWS_ACCESS_KEY_ID     = credentials('AKIA4Z5W7B6HQNT34PS5')
        AWS_SECRET_ACCESS_KEY = credentials('svNJ5v+ul61KCIM8P//ek3WIiYJEs0MgeNDGQAj2')
        AWS_DEFAULT_REGION    = 'ap-south-1'  // Change to your desired region
        ECR_REPO_NAME         = 'serverless' // Change to your ECR repository name
        DOCKER_IMAGE_NAME     = 'buildfile' // Change to your desired image name
        DOCKERFILE_PATH       = 'D:/jenkins-cluster-on-aws/react-app/react/Dockerfile.build' // Path to your Dockerfile
    }

    node('workers'){
    stage('Checkout'){
        checkout scm
    }

    stage('Test'){
        sh "echo 'running tests'"
    }

    stage('Build'){
        sh """
            docker build -f Dockerfile.build -t buildfile .
            containerName=\$(docker run -d buildfile)
            docker cp \$containerName:/app/build buildfile
            docker rm -f \$containerName
            zip -r deployment.zip buildfile
        """ 
    }

     stage('Push to ECR') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'AWS_ACCESS_KEY_ID', variable: 'AWS_ACCESS_KEY_ID'),
                                     string(credentialsId: 'AWS_SECRET_ACCESS_KEY', variable: 'AWS_SECRET_ACCESS_KEY')]) {
                        sh """
                            $(aws ecr get-login --no-include-email --region ${AWS_DEFAULT_REGION})
                            docker tag ${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER} ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${ECR_REPO_NAME}:${env.BUILD_NUMBER}
                            docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${ECR_REPO_NAME}:${env.BUILD_NUMBER}
                        """
                    }
                }
            }
        }
    
    post {
        always {
            script {
                // Clean up Docker images if needed
                sh "docker rmi ${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}"
            }
        }
    }


    // stage('Push to s3'){
    //     sh "echo script running"
    //     if (env.BRANCH_NAME == 'devlop' || env.BRANCH_NAME == 'prepod' || env.BRANCH_NAME == 'master')
    //     sh "aws s3 cp deployment.zip s3://${bucket}/${functionName}/${environments[env.BRANCH_NAME]}/"
    //  }

    // stage('Deploy'){
    //     if(env.BRANCH_NAME == 'devlop' || env.BRANCH_NAME == 'prepod' || env.BRANCH_NAME == 'master'){
    //         sh """
    //             aws lambda update-function-code --function-name ${functionName} --s3-bucket ${bucket} --s3-key ${functionName}/${environments[env.BRANCH_NAME]}/deployment.zip --region ${region}
    //             version=\$(aws lambda get-alias --function-name ${functionName} --name ${environments[env.BRANCH_NAME]} --region ${region} | jq -r '.FunctionVersion')
    //             new_envvars=\$(aws lambda get-function-configuration --function-name ${functionName} --region ${region} --qualifier \$version --query "Environment.Variables") 
    //              aws lambda update-function-configuration --function-name ${functionName} --environment "{ \\"Variables\\": \$new_envvars }" --region ${region}
    //         """
            
    //         sleep(3)
            
    //         sh """
    //             publishedVersion=\$(aws lambda publish-version --function-name ${functionName} --description ${environments[env.BRANCH_NAME]} --region ${region} | jq -r '.Version')
    //             aws lambda update-alias --function-name ${functionName} --name ${environments[env.BRANCH_NAME]} --function-version \$publishedVersion --region ${region}
    //         """
    //     }
    // }
    }

}