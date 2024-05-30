# Flask WEB API Example

## Create a AWS EC2 instance

Please flollow the instructions that Paul provided to us to create an AWS EC2 instance. We will get the instance public IP address and use it to access the web API. The port 80 by default is open for the instance.

## AWS setup

`ssh` to your EC2 instance. (If you are not familiar with `ssh`, please just click the `Connect` button on the top right corner of EC2 instance page and select the tab `EC2 Instance Connect` by default, you will be able to connect to the instance without any further setup.)

![AWS connection](./images/aws-conn.jpg)

### Set flask app environment

1. create a `app` folder in the home directory of the instance.
2. put the `app.py` file in the `app` folder. The file exampe is avaliable in [this repository](https://github.com/Data472-Individual-Project-Pipeline/flask-web-api-example).
3. create the `venv` folder in the `app` folder.
4. make the current shell use the venv folder by running the following command:

```bash
sudo apt-get update
sudo apt install python3-virtualenv
virtualenv -p python3 venv
source venv/bin/activate
```

5. install the required packages and freeze the requirements:

```bash
pip install flask
pip install gunicorn
pip freeze > requirements.txt
```

6. Finally, your app folder should be like the following:

![Files structure](./images/aws-files.jpg)
 
```bash
(venv) ubuntu@ip-10-0-1-217:~/app$ tree -L 2
.
├── app.py
├── requirements.txt
└── venv
    ├── bin
    ├── lib
    └── pyvenv.cfg
```

1. now you can try to run the flask app by running the following command `python app.py`. You should be able to see the following output:

```bash
 * Running on all addresses (0.0.0.0)
 * Running on http://127.0.0.1:8000
 * Running on http://10.0.1.217:8000
Press CTRL+C to quit
```

### Set gunicorn service

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

![Gunicorn Config](./images/services.jpg)

1. enable the service by running the following command:

```bash
sudo systemctl start flaskapp
sudo systemctl enable flaskapp
```

now the flask app should be running as a service. Testing it by type `curl http://localhost:8000` in the shell. You should be able to see the following output:

```bash
{
  "message": "Your json data response here...!"
}
```

### Set Nginx

1. install nginx by running the following command:

```bash
sudo apt-get update
sudo apt-get install nginx
```

2. start Nginx:

```bash
sudo systemctl start nginx
```

1. enable nginx reverse proxy by updating a file named `default` in the `/etc/nginx/sites-available/` folder. The file finally should be like the following: the IP address `3.25.86.51` is my AWS EC2 instance public IP address, you should replace it with your own IP address.

![Nginx config](./images/nginx.jpg)

```bash
server {
        listen 80 default_server;
        listen [::]:80 default_server;

        root /var/www/html;
        index index.html index.htm index.nginx-debian.html;

        server_name 3.25.86.51;

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

1. restart Nginx by running the following command:

```bash
sudo systemctl restart nginx
```

![Two services](./images/ser2.jpg)

```JavaScript
[
    {
    "aqi": 50,
    "co": 60,
    "date": "2024-05-1",
    "humidity": 50,
    "location": "Location 1",
    "no2": 40,
    "o3": 30,
    "pm10": 20,
    "pm25": 10,
    "region": "Region 1",
    "so2": 50,
    "temperature": 20,
    "wind_speed": 5
    },
    {
    "aqi": 51,
    "co": 61,
    "date": "2024-05-2",
    "humidity": 51,
    "location": "Location 2",
    "no2": 41,
    "o3": 31,
    "pm10": 21,
    "pm25": 11,
    "region": "Region 2",
    "so2": 51,
    "temperature": 21,
    "wind_speed": 6
    },
    {
    "aqi": 52,
    "co": 62,
    "date": "2024-05-3",
    "humidity": 52,
    "location": "Location 3",
    "no2": 42,
    "o3": 32,
    "pm10": 22,
    "pm25": 12,
    "region": "Region 3",
    "so2": 52,
    "temperature": 22,
    "wind_speed": 7
    }
]
```

now you should be able to access the web API by using the public IP address of the EC2 instance. Donot use https, just use http. The AWS EC2 instance I used is called `i-0500e178a5e5e1778 (DATA472-hwa205-flaskwebapiexample)` You can go AWS to access it to check any configuration files I mentioned above.

![My AWS EC2 instance](./images/aws.jpg)

## Conclusion

There are so many ways to deploy a web api application to AWS EC2 Instance, such as using Docker, Apache, Nginx, etc. The above example is just one of the ways. I hope this example can help you to understand how to deploy a web api application to AWS EC2 Instance. The only things you need to do is that just write a data pipeline ETL code to your python script. If you have any questions, please feel free to ask me or Jacob or contact Central Collection Team.
