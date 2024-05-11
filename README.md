# Flask WEB API Example

## Create a AWS EC2 instance

Please flollow the instructions that Paul provided to us to create an AWS EC2 instance. We will get the instance public IP address and use it to access the web API. The port 80 by default is open for the instance.

## AWS setup

SSH to your EC2 instance. (If you are not familiar with SSH, please just click the `Connect` button on the top right corner of EC2 instance page and select the tab `EC2 Instance Connect` by default, you will be able to connect to the instance without any further setup.)

### set flask app environment

1. create a `app` folder in the home directory of the instance.
2. put the `app.py` file in the `app` folder. The file exampe is avaliable in [this repository](https://github.com/Data472-Individual-Project-Pipeline/flask-web-api-example).
3. create the `venv` folder in the `app` folder.
4. make the current shell use the venv folder by running the following command:

```bash
virtualenv -p python3 venv
source app/venv/bin/activate
```

5. install the required packages and freeze the requirements:

```bash
pip install flask
pip install gunicorn
pip freeze > requirements.txt
```

6. now you can try to run the flask app by running the following command `python app.py`. You should be able to see the following output:

```bash
* Running on http://127.0.0.1:8080/ (Press CTRL+C to quit)
```

### set gunicorn service

1. create a `flaskapp.service` file in the `/etc/systemd/system/` folder. The file content should be like the following:

```bash
[Unit]
Description=Data472 individual project flask app
After=network.target
[Service]
User=ubuntu
Group=ubuntu
WorkingDirectory=/home/ubuntu/app
ExecStart=/home/ubuntu/app/venv/bin/gunicorn app
[Install]
WantedBy=multi-user.target
```

notes: If your EC2 instance image is not ubuntu, you should replace the `ubuntu` with the correct user name.

2. enable the service by running the following command:

```bash
sudo systemctl start flaskapp
sudo systemctl enable flaskapp
```

now the flask app should be running as a service.

### set nginx

1. install nginx by running the following command:

```bash
sudo apt-get update
sudo apt-get install nginx
```

2. start Nginx:

```bash
sudo systemctl start nginx
```

3. enable nginx reverse proxy by creating a file named `flaskapp.conf` in the `/etc/nginx/conf.d/` folder. The file content should be like the following:

```bash
server {
    listen 80 default_server;
    server_name _; # replace underline with your aws instance public IP address
    access_log  /var/log/nginx/access.log;
    error_log  /var/log/nginx/error.log;

    location / {
        proxy_pass         http://127.0.0.1:8000/;
        proxy_redirect     off;

        proxy_set_header   Host                 $host;
        proxy_set_header   X-Real-IP            $remote_addr;
        proxy_set_header   X-Forwarded-For      $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarded-Proto    $scheme;
    }
}
```

4. restart nginx by running the following command:

```bash
sudo systemctl restart nginx
```

now you should be able to access the web API by using the public IP address of the EC2 instance. Donot use https, just use http.