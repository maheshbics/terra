def region = 'ap-south-1'
def dockerimagename = 'buildfile'
def awsaccountid ='880315142031'
def ecrreponame = 'terra-react'
def ec2sshKey = 'C:/Users/YOGA/Downloads/id_rsa'
def ecrRegion = 'ap-south-1'
def ecrRepoUri = '880315142031.dkr.ecr.ap-south-1.amazonaws.com/terra-react'
def contname = 'C7'
def localFilePath = 'C:/Users/YOGA/Downloads/id_rsa'
def Remoteuser = 'ec2-user'
def ec2InstanceIp = '3.110.162.54'
def remoteHostPath = '/var/lib/docker'
def AcessKey = 'AKIA4Z5W7B6HQNT34PS5'
def secretKey = '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'

pipeline {
    agent any
    stages{
        stage('Checkout'){
            agent {
                label 'workers'
            }
            steps{
                checkout scm
            }
        }

        stage('Test'){
            agent {
                label 'workers'
            }
       
            steps{
                sh "echo 'running tests'"
            }
        }

        stage('Build'){
            agent {
                label 'workers'
            }
            steps{
                sh """
                    docker build -f Dockerfile.build -t buildfile .
                    containerName=\$(docker run -d buildfile)
                    docker cp \$containerName:/app/build buildfile
                    docker rm -f \$containerName
                    zip -r deployment.zip buildfile
            
                """ 
            }
        }
        

        stage('Push to ECR') {
            agent {
                label 'workers'
            }
            steps{
                sh """
                    aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${awsaccountid}.dkr.ecr.${region}.amazonaws.com 
                    aws ecr describe-repositories --repository-names ${ecrreponame} --region ${region} || aws ecr create-repository --repository-name ${ecrreponame} --region ${region}
                    docker tag ${dockerimagename} ${awsaccountid}.dkr.ecr.${region}.amazonaws.com/${ecrreponame}:${env.BUILD_NUMBER}
                    docker push ${awsaccountid}.dkr.ecr.${region}.amazonaws.com/${ecrreponame}:${env.BUILD_NUMBER}
                """  
            }   
        }

        stage('deploy on EC2') {
            steps{
                sshagent(['3.110.162.54']){
                    sh "ssh ec2-user@3.110.162.54 'aws configure set aws_access_key_id ${AcessKey}'"
                    sh "ssh ec2-user@3.110.162.54 'aws configure set aws_secret_access_key ${secretKey}'"
                    sh "ssh ec2-user@3.110.162.54 'aws configure set default.region ${region}'"
                    sh "ssh ec2-user@3.110.162.54 'aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${awsaccountid}.dkr.ecr.${region}.amazonaws.com'"
                    sh "ssh ec2-user@3.110.162.54 'docker pull ${ecrRepoUri}:${env.BUILD_NUMBER}'"
                    sh "ssh ec2-user@3.110.162.54 'docker stop ${contname} || true && docker rm ${contname} || true'"
                    sh "ssh ec2-user@3.110.162.54 'docker run -itd --name ${contname} -p 3000:3000 ${ecrRepoUri}:${env.BUILD_NUMBER}'"
                }
            }
    
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

    

}
