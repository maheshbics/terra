def region = 'ap-south-1'
def dockerimagename = 'buildfile'
def awsaccountid ='880315142031'
def ecrreponame = 'terra-react'


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
        sh """
            aws ecr get-login --no-include-email --region ${region}
            docker tag ${dockerimagename}:${env.BUILD_NUMBER} ${awsaccountid}.dkr.ecr.${region}.amazonaws.com/${ecrreponame}:${env.BUILD_NUMBER}
            docker push ${awsaccountid}.dkr.ecr.${region}.amazonaws.com/${ecrreponame}:${env.BUILD_NUMBER}
        """     
    }
    
    // post {
    //     always {
    //         script {
    //             // Clean up Docker images if needed
    //             sh "docker rmi ${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}"
    //         }
    //     }
    // }


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