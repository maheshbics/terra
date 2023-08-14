// def bucket = 'oraculi-terraform-states7'
// def functionName = 'serverless'
// def region = 'ap-south-1'
// def environments = ['devlop': 'sandbox', 'prepod': 'staging', 'master': 'production']
// def s3Uri = 's3://${bucket}/${functionName}/${environments[env.BRANCH_NAME]}/'


node('workers'){
    stage('Checkout'){
        checkout scm
    }

    stage('Test'){
        sh "echo 'running tests'"
    }

    stage('Build'){
        sh """
            docker build -f Dockerfile.build -t serverless-app .
            containerName=\$(docker run -d serverless-app)
            docker cp \$containerName:/go/src/github.com/maheshbics/lambda-serverless/main main
            docker rm -f \$containerName
            zip -r deployment.zip main
        """ 
    }

    stage('Push'){
        sh "echo script running"
        if (env.BRANCH_NAME == 'devlop' || env.BRANCH_NAME == 'prepod' || env.BRANCH_NAME == 'master')
        sh "aws s3 cp main s3://${bucket}/${functionName}/${environments[env.BRANCH_NAME]}/"
     }

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