
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
