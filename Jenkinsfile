#!groovy

node {

  stage 'Checkout'
    checkout scm

  stage 'Install packer'
    sh "curl -o packer.zip https://releases.hashicorp.com/packer/1.1.3/packer_1.1.3_linux_amd64.zip?_ga=2.25536829.1444204868.1515428339-1209176675.1503914009"
    sh "unzip -o packer.zip"

  stage 'Validate'
    def packer_file = 'packer.json'
    print "Running packer validate on : ${packer_file}"
    sh "./packer -v ; ./packer validate ${packer_file}"

  stage 'Build'
    sh "./packer build ${packer_file}"

  stage 'Get AMI ID'
    ami = sh(returnStdout: true, script: "grep artifact manifest.json | tail -1 | awk -F':' '{print \$3}' | sed 's/\",//g'").trim()
    print "Here is the AMI - ${ami}"

  stage 'Create Cloudformation'
    sh "aws cloudformation create-stack --stack-name myteststack --template-body file://frontend.yaml --parameters ParameterKey=Ami,ParameterValue=${ami} --capabilities CAPABILITY_IAM --region us-east-2"

  stage 'Status'
    sh "aws cloudformation wait stack-create-complete --stack-name myteststack --region us-east-2"
}
