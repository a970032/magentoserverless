# README #  
  
This README would normally document whatever steps are necessary to get your application up and running.  
  
## What is this repository for? ##  
  
Build Magento in serverless way  
  
## How do I get set up? ##  
  
Create EFS and a ec2(AMI)
Create EFS accesspoint and mount to ec2  
ssh into the ec2 and pull this project  
  
  
### In magento folder ###  
Install the magento project in magento folder with bellow command: 
 
export COMPOSER_PROCESS_TIMEOUT=6000  
apt-get update  
composer update --ignore-platform-reqs --update-no-dev  
bin/magento module:enable -all  
bin/magento setup:di:compile  
bin/magento setup:static-content:deploy --no-parent  
  
copy the pub/static to <EFS>/pub/static  
copy the pub/media to <EFS>/pub/media  
Create a symlink for var folder to <EFS>/var  
  
Install the magento:  
bin/magento setup:install \  
--base-url=<URL> \  
--db-host=<DB URL> \  
--db-name=magento \  
--db-user=admin \  
--db-password=Abcd1234% \  
--admin-firstname=admin \  
--admin-lastname=admin \  
--admin-email=admin@admin.com \  
--admin-user=admin \  
--admin-password=admin123 \  
--language=en_US \  
--currency=USD \  
--timezone=America/Chicago \  
--use-rewrites=1 \  
--search-engine=opensearch \  
--opensearch-host=<OPEN SEARCH URL> \  
--opensearch-port=443 \  
--opensearch-index-prefix=magento \  
--opensearch-timeout=15  
  
### In magentoserverless folder ###  
composer update --ignore-platform-reqs --update-no-dev  
config the ARN,VPC,SG with yours in serverless.yml  
Install the serverless command:  
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/master/install.sh | bash  
//close shell and open again  
npm install -g serverless@3  
  
serverless deploy  
  
Then in CloudFormation you should see related resource is created  
After that update the magento base url with the apigatway url  
  
  
### EFS access policy ###  
  
  
{  
    "Version": "2012-10-17",  
    "Id": "efs-policy-wizard-4d32d9db-a52e-40c1-8560-328280f9d2e6",  
    "Statement": [  
        {  
            "Sid": "efs-statement-4316eae8-36e9-4351-99c0-b2faef891431",  
            "Effect": "Allow",  
            "Principal": {  
                "AWS": "*"  
            },  
            "Action": [  
                "elasticfilesystem:ClientMount",  
                "elasticfilesystem:ClientWrite",  
                "elasticfilesystem:ClientRootAccess",  
                "elasticfilesystem:DescribeMountTargets"  
            ],  
            "Resource": "arn:aws:elasticfilesystem:ap-east-1:039092478854:file-system/fs-06ec91843bcfc2417",  
            "Condition": {  
                "Bool": {  
                    "elasticfilesystem:AccessedViaMountTarget": "true"  
                }  
            }  
        }  
    ]  
}  
  
### Optional ###  
  
You can merge and minify the css/js to speed up the static content loading  
  
config:set dev/css/merge_css_files 1  
config:set dev/css/minify_files 1  
config:set dev/js/merge_files 1  
config:set dev/js/enable_js_bundling 1  
config:set dev/template/minify_html 1  
config:set dev/static/sign 0  
  
You can set up a simple apache server to serve all the static content and media by pointing the pub/static pub/media folder to the ec2  
As they stored in EFS

To speed up the site you can setup some cron job to keep the lambda function warm
For example:
Keep sending request to the site every 5 minute so that the cold start can be avoided