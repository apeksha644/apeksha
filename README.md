# Project Name : Flask Project
# Created By : Mr. Nagesh Patil
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

1) Steps to set up and deploy the application.

a.	Create a simple Python Flask application that returns "Hello World!".

•	Create Virtual Environment:
    Install virtualenv using pip command
    $ pip3 install virtualenv
    $ virtualenv env
    $ source env/bin/activate
    •	Set up the Flask application to run with Gunicorn as the WSGI server.
    Install gunicorn & flask: 
    $ pip install gunicorn
    $ pip install flask
    •	Create flask application file app.py.
    app.py:
    from flask import Flask
    app = Flask(__name__)
    @app.route("/")
    def index():
        return "Hello World"
    if __name__ == "__main__":
        app.run(host='0.0.0.0', port=8080)

b.	Set up the Flask application to run with Gunicorn as the WSGI server.

•	Create WSGI configuration for gunicorn interact with application.
    wsgi.py:

    from app import app
    if __name__ == "__main__":
        app.run()

•	Once wsgi file is ready then we are serving our application using below command.
    $ gunicorn --bind 0.0.0.0:5000 wsgi:app

    ![image](https://github.com/NageshPatil321/flaskproject/assets/63147214/b7c99429-b40d-46c7-9b3d-982e0729ae9a)

•	Now our application is running but when we stop or cancel that time application will be stop.

    ![image](https://github.com/NageshPatil321/flaskproject/assets/63147214/ee853f74-0445-430e-ae12-1004d1bb5337)

•	To overcome this issue, we need to create a gunicorn systemd service and this service will be managing the application like start, stop, restart etc.
    $ vim /etc/systemd/system/gunicorn.service
 
    ![image](https://github.com/NageshPatil321/flaskproject/assets/63147214/c4ac17de-01a0-4876-a361-36f06d22f5b8)

c.	Configure Nginx as a reverse proxy to forward requests to the Gunicorn server.

•	Now we need to configure our application site configuration inside the nginx server to serving application via gunicorn.

    $ cd /etc/nginx/site-available
    $ vi application.com # create file for our application configuration details 
    $ ln -s /etc/nginx/sites-available/application.com /etc/nginx/sites-enabled # enable config file 

    ![image](https://github.com/NageshPatil321/flaskproject/assets/63147214/3d53b899-3bb3-42be-a76d-0032e3a3433e)

"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

2) CI/CD pipeline configuration.

•	Add the web application code to the repository.

    ![image](https://github.com/NageshPatil321/flaskproject/assets/63147214/09be0aee-73b2-4db9-b559-f1e2698d3e37)

•	Create a Git repository on GitHub (or another Git hosting service).

    ![image](https://github.com/NageshPatil321/flaskproject/assets/63147214/91386a5f-90df-4c4c-ae63-f225161e3454)

•	Create a CI/CD pipeline using GitHub Actions (or any other CI/CD tool like Jenkins, GitLab CI/CD). The Jeniks setup and pipeline should:

    ![image](https://github.com/NageshPatil321/flaskproject/assets/63147214/3b7eba05-8bd1-449a-a133-78409ceb5a10)

        Pipeline Script:
        pipeline {
            agent any
            environment {
                VENV_DIR = 'venv'
            }
            stages {
                stage('Clone Repository') {
                    steps {
                        git branch: 'main', credentialsId: 'GitHub-Server-ID', url: 'https://github.com/NageshPatil321/flaskproject.git'
                    }
                }
                stage('Setup Python Environment') {
                    steps {
                        script {
                            // Install virtual environment and dependencies
                            sh '''
                                python3 -m venv $VENV_DIR
                                . $VENV_DIR/bin/activate
                                pip install --upgrade pip
                                pip install -r requirements.txt
                            '''
                        }
                    }
                }
                stage('Deploy') {
                    steps {
                        sh 'cp -rf /var/lib/jenkins/workspace/flask-pipeline/* /opt/myFlaskApplication/'
                        sh 'zip -r /var/lib/jenkins/workspace/flask-pipeline/artifact.zip /var/lib/jenkins/workspace/flask-pipeline/'
                    }
                }



                stage('Application Reload') {
                    steps {
                        sh 'sudo systemctl restart gunicorn'
                        sh 'sudo systemctl restart nginx'
                    }
                }
                stage('Backup to S3') {
                    steps {
                        script {
                            // Upload the zip file to S3 using the S3 Plugin
                            s3Upload consoleLogLevel: 'INFO',
                                    dontSetBuildResultOnFailure: false,
                                    dontWaitForConcurrentBuildCompletion: false,
                                    entries: [[
                                        bucket: 'devops-assignment-nagesh-patil',
                                        gzipFiles: false,
                                        keepForever: false,
                                        managedArtifacts: false,
                                        noUploadOnFailure: true,
                                        selectedRegion: 'ca-central-1',
                                        showDirectlyInBrowser: false,
                                        sourceFile: "**/artifact.zip", // Ant GLOB pattern
                                        storageClass: 'STANDARD',
                                        uploadFromSlave: false,
                                        useServerSideEncryption: false
                                    ]],
                                    pluginFailureResultConstraint: 'FAILURE',
                                    profileName: 'S3-Bucket-Storage',
                                    userMetadata: []
                        }
                    }
                }
                stage('clean WS') {
                    steps {
                        cleanWs()
                    }
                }
            }
        }


•	Jenkins Build Status

    ![image](https://github.com/NageshPatil321/flaskproject/assets/63147214/1e18263f-a13c-4c0b-83b2-15ee02a9c9c7)

•	Application running status

    ![image](https://github.com/NageshPatil321/flaskproject/assets/63147214/49d8bc27-daad-46b8-86ab-29f1cb1d2f43)

•	Deploy the updated application to the EC2 instance

    ![image](https://github.com/NageshPatil321/flaskproject/assets/63147214/49de2bb9-04e2-4fed-857c-9aa292379154)

    ![image](https://github.com/NageshPatil321/flaskproject/assets/63147214/477de2d9-5844-4f35-abb7-7bcd8c627645)

•	Update the S3 bucket with any static files from the application

    ![image](https://github.com/NageshPatil321/flaskproject/assets/63147214/48ee8d2a-d302-4f0c-9ff2-c9f7ab2f69d6)

"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

3) IAM roles and policies configuration

•	Set up IAM roles and policies to ensure the EC2 instance can interact with the S3 bucket and Attach created role to the EC2 instance

    ![image](https://github.com/NageshPatil321/flaskproject/assets/63147214/868f27f4-c866-4c77-86c3-e7ab712a26c4)

    poilicy attach:

    ![image](https://github.com/NageshPatil321/flaskproject/assets/63147214/d4c300de-fff8-4c35-bbc8-485a526eb679)

•	Ensure proper security groups are configured for the EC2 instance to allow web traffic on port 80.

    ![image](https://github.com/NageshPatil321/flaskproject/assets/63147214/de7f26c9-7f23-49e6-a327-2b7b3159c287)

•	Set up a schedule to start and stop the EC2 instance at specified times using AWS Lambda and CloudWatch Events.

    a) Create Policy for EC2 Instance to Start and Stop the Servers.

    ![image](https://github.com/NageshPatil321/flaskproject/assets/63147214/ff29d20b-b625-481f-99d0-b23570f61205)

    b) Create Role for Lambda Function to start and Stop and attach this above policy.

    ![image](https://github.com/NageshPatil321/flaskproject/assets/63147214/3093df45-bb53-488f-87e3-cb29eab7ddfe)

    ![image](https://github.com/NageshPatil321/flaskproject/assets/63147214/8e6304db-46b2-432d-acca-96b1449a9031)

    c) Create Lambda Function for Start and Stop the Server.

    ![image](https://github.com/NageshPatil321/flaskproject/assets/63147214/2b3e5c71-690b-45a1-a52d-fedb275c7cc9)

    d) AWSPolicyFlaskStartServer-Lambda Python Code.

    ![image](https://github.com/NageshPatil321/flaskproject/assets/63147214/216fd522-dff0-4b5e-b072-ad43bf3d6f54)

    e) AWSPolicyFlaskStopServer-Lambda Python Code.

    ![image](https://github.com/NageshPatil321/flaskproject/assets/63147214/1112a366-023a-477c-85fc-e44d48ea5f41)

    f) CloudWatch Setup.

    Start:

    ![image](https://github.com/NageshPatil321/flaskproject/assets/63147214/06478d6c-a4c6-4897-a734-c31529dab519)

    Stop:

    ![image](https://github.com/NageshPatil321/flaskproject/assets/63147214/e84fc0fa-6736-4292-bc30-ff52258d8daf)


"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
4) Configure S3 for Static File Hosting

•	Create an S3 bucket named devops-assignment-[your_name].

    ![image](https://github.com/NageshPatil321/flaskproject/assets/63147214/986e1aa5-12cd-443f-80c2-aec32bd04317)

•   Upload a file (e.g., a text file with a brief introduction about yourself) to the S3 bucket.

    ![image](https://github.com/NageshPatil321/flaskproject/assets/63147214/67be87d1-d4d1-4be0-afe9-ff64256f062f)

•	Set appropriate permissions to ensure the file is publicly accessible.

    1)	Static File Hosting:

    URL : http://devops-assignment-nagesh-patil.s3-website.ca-central-1.amazonaws.com/ 

    Screenshot:

    ![image](https://github.com/NageshPatil321/flaskproject/assets/63147214/2c2dd49b-75b9-4324-9081-3d63ce7ebc8f)

    2)	introduction.txt file publicly accessible   

    URL : https://devops-assignment-nagesh-patil.s3.ca-central-1.amazonaws.com/introduction.txt 
 
    Screenshot:

    ![image](https://github.com/NageshPatil321/flaskproject/assets/63147214/9b60d9f5-8f9f-4f22-b10b-17a81a3cf0bd)

"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
5) Cron job setup

•	Write a cron job script on the EC2 instance to check the application health (e.g., make an HTTP request to the web app) and log the status every 5 minutes.

    a) Write the Application Health Check Shell Script

    ![image](https://github.com/NageshPatil321/flaskproject/assets/63147214/75881e66-25d1-4834-b5aa-04fb640d05a1)

    b) Set the crontab function inside the server using below command
     $ crontab -e 
     
     Then we are set the timing according to our requirement so here we are set time every 5 min with shell script file location.
     
     */5 * * * * /usr/local/bin/flask_app_health.sh

    C) Check the application log the status every 5 minutes using crontab

    ![image](https://github.com/NageshPatil321/flaskproject/assets/63147214/181769ab-ed4b-470b-acdc-7e57fb742257)

"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

6) Any issues faced and how you resolved them

    a) Getting "error: externally-managed-environment" while installing pip related command.

    Solution : To resolve this issue with parse the "--break-system-packages" and install the package using pip.
    Example : pip install gunicorn --break-system-packages

    b) After successfully configured all the configuration services but still getting error 502 bad gateway.

    Solution : Application is not running after proper configuration and getting error 502 bad gateway then i will check the logs of nginx error.log inside the "/var/log/nginx/" and error showing permission denid so i have given required permission of the directory using chmod 775 command and then application will be accessible.

    c) After EC2 scheduling EC2 instance public ip will be change and again need to update the public ip inside the config file.

    Solution : Assign Elastic IP address to EC2 instance and update that public IP inside the config files.

"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
