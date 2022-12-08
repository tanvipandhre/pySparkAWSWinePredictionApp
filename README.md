# AWS Spark Wine Quality Prediction Application

AWSSPARKWINEAPP is a pyspark appliaction. The purpose of this project is to train a machine learning model parallely on ec2 instances for predicting wine quality on a piblically available data and then use the trained model to predict the wine quality.  

Project also uses Docker to create a container for trained machine learning model to simplify deployments.
 
This project contains 2 main python source files:
 
`wine_prediction.py` reads training dataset from S3 and trains model in parallel on EMR spark cluster. Once model is trained, it can be run on provided test data provided via S3. This program stores trained model in S3 bucket (Public URL for bucket - S3://wine-data-12)

`wine_test_data_prediction.py` program loads trained model and executes that model on given testdata file. This 
will then print F1 score as metrics for accuracy of the trained model.

Dockerfile: Dockerfile to create docker image and run container for simplified deployment.

##  Instruction to use:
### 1. How to create Spark cluster in AWS 
User can create spark cluster using EMR console provided by AWS. Please follow steps to create one with 4 ec2 instances (use more instances depending upon your load).

1. Create Key-Pair for EMR cluster using navigation ```EC2-> Network & Security -> Key-pairs```.
   Use .pem as format. This will download {name of key pair}>.pem file. Keep it safe you will need that to do SSH to EC2 instances.
2. Navigate to Amazon EMR console using link  https://console.aws.amazon.com/elasticmapreduce/home?region=us-east-1. Then, navigate to clusters-> create cluster.
3. Now fill in respective sections:
```
   General Configuratin -> Cluster Name 
   Software Configuration-> EMR 5.33 , do select 'Spark: Spark 2.4.7 on Hadoop 2.10.1 YARN and Zeppelin 0.9.0' option menu.
   Harware Configuration -> Make instance count as 4
   Security Access -> Provide .pem key created in above step.
   Rest of parameters can be left default.
 ```

User can also create spark cluster using below aws cli command:
  ```
  aws emr create-cluster --applications Name=Spark Name=Zeppelin --ebs-root-volume-size 10 --ec2-attributes '{"KeyName":"ec2-spark","InstanceProfile":"EMR_EC2_DefaultRole","SubnetId":"subnet-42c0ca0f","EmrManagedSlaveSecurityGroup":"sg-0d7ed2552ba71f5af","EmrManagedMasterSecurityGroup":"sg-0e853f0a4bdc5f799"}' --service-role EMR_DefaultRole --enable-debugging --release-label emr-5.33.0 --log-uri 's3n://aws-logs-367626191020-us-east-1/elasticmapreduce/' --name 'My cluster' --instance-groups '[{"InstanceCount":3,"EbsConfiguration":{"EbsBlockDeviceConfigs":[{"VolumeSpecification":{"SizeInGB":32,"VolumeType":"gp2"},"VolumesPerInstance":2}]},"InstanceGroupType":"CORE","InstanceType":"m5.xlarge","Name":"Core Instance Group"},{"InstanceCount":1,"EbsConfiguration":{"EbsBlockDeviceConfigs":[{"VolumeSpecification":{"SizeInGB":32,"VolumeType":"gp2"},"VolumesPerInstance":2}]},"InstanceGroupType":"MASTER","InstanceType":"m5.xlarge","Name":"Master Instance Group"}]' --scale-down-behavior TERMINATE_AT_TASK_COMPLETION --region us-east-1
  ```
  
4. Cluster status should be 'Waiting' on successful cluster creation.

### 2. How to train ML model in Spark cluster with 4 ec2 instances in parallel
1. Now when cluster is ready to accept jobs, to submit one you can either use step button to add steps or submit manually.
   To submit manually, Perform SSH to Master of cluster using below command:
```
        ssh -i "ec2key.pem" <<User>>@<<Public IPv4 DNS>>
        
```
2. On successful login to master , change to root user by running command:
  ```
  sudo su
  ```
3. Submit job using following command:
 ```
   spark-submit s3://wine-data-12/wine_prediction.py
 ```
4. You can trace status of this job in EMR UI application logs. Once status is succeded a test.model will be created in s3 bucket-s3://wine-data-12.


### 3. How to run trained ML model locally without docker.
1. Clone this repository.
2. Make sure you have spark environment setup locally for running this. To setup one follow link https://spark.apache.org/docs/latest
3. Navigate to awssparkwineappk/src folder
4. Place your testdata in 'awssparkwineappk/data/csv' folder
5. Install pyspark, you can use pip -m install pyspark. Or install `` conda`` package.
6. Once setup is ready execute below command:
   Execute following command
 ``` 
 cd awssparkwineapp/src
 spark-submit wine_test_data_prediction.py <filename>
 ```
 
### 4. Run ML model using Docker
1. Install docker where you want to run this container
2. A public image has been created and posted on DockerHub. Use the command **Docker pull dfordeepika/awssparkwineapp** to get the image on your machine.
3. Place your testdata file in a folder (lets call it directory dirA) , which you will mount with docker container.
4. docker run -v {directory path for data dirA}:/code/data/csv dfordeepika/awssparkwineapp {testdata file name}

Sample command
```
docker run -v /Users/<username>/<path-to-folder>/awssparkwineapp/data/csv:/code/data/csv dfordeepika/awssparkwineapp testdata.csv

```
