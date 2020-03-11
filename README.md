# AWS nlb-lambda for Python3 
# THANK YOU https://bezdelev.com/hacking/aws-cli-inside-lambda-layer-aws-s3-sync/
## Improvements
* Python 3.7 support
* Capability to extract internal IPs from internet-facing ALB

## Notes
Uses Lambda Layer to leverage awscli in py3 environment

# setup

## prepare dependencies
```
stack_name="py3"
nlb_lambda_bucket_name=$(echo "${stack_name}-nlblambda-code" | tr '[:upper:]' '[:lower:]')
git clone https://github.com/ilyabezdelev/aws-cli-lambda.git
cd aws-cli-lambda/
```
### you may need to run the "linux" shell script dependent on your OS.
```
chmod +x awscli-lambda-package_macos.sh && ./awscli-lambda-package_macos.sh
```
## create s3 bucket to host the script
```
aws s3api create-bucket --bucket ${nlb_lambda_bucket_name} 
zip nlb-lambda-py3.zip nlb-lambda.py
```
## store needed files
```
aws s3 cp ./nlb-lambda-py3.zip s3://${nlb_lambda_bucket_name}/code/ 
aws s3 cp ./aws-cli-lambda/awscli-lambda-layer.zip s3://${nlb_lambda_bucket_name}/code/ 
```

## Nested CloudFormation Template
```
NLBLambdaStack:
    Type: 'AWS::CloudFormation::Stack'
    Properties:
      TemplateURL: 'cfn-nlb-lambda.json'
      Parameters:
        NLBTargetGroupARN: ''
        InternalALBDNSName: ''
        S3BucketName: ''
        ALBListenerPort: 443
        LambdaS3Bucket: 'bucketName'
        LambdaS3Key: 'code/nlb-lambda-py3.zip'
        LambdaLayerS3Key: 'code/awscli-lambda-layer.zip'
```


# development

## test locally

python-lambda-local -l aws-cli-lambda/ -f lambda_handler -e vars.json nlb-lambda.py

