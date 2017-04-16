## dsserver

In this post I show how to setput a server for data science project. I will install the following tools: 
- Python (Anaconda distributio)
- Jupyter
- Rstudio
- Keras
- Theano
- Tensoflow
- Xgboost


> Update  
```sh
# update package source
sudo apt-get update

# upgrade package
sudo apt-get upgrade
``` 

> step 1: install Python Anaconda into /usr/local/anaconda2

```sh  
#  download package
wget https://3230d63b5fc54e62148e-c95ac804525aac4b6dba79b00b39d1d3.ssl.cf1.rackcdn.com/Anaconda2-4.0.0-Linux-x86_64.sh

# Install package
bash Anaconda2-4.0.0-Linux-x86_64.sh
``` 

> step 2: install and configure jupyter  

```sh  
# 1- Install pip3 and other dependencies

sudo add-apt-repository ppa:fkrull/deadsnakes
sudo apt-get update
sudo apt-get install python3.5

wget https://bootstrap.pypa.io/get-pip.py

chmod 777 get-pip.py

sudo /usr/bin/python3.5 get-pip.py

sudo apt-get install npm 
sudo apt-get install nodejs-legacy
sudo npm install -g configurable-http-proxy

# 2- install JupyterHub and Jupyter for python 3 kernel

sudo /usr/bin/python3.5 -m pip install jupyterhub
sudo /usr/bin/python3.5 -m pip install "ipython[notebook]"
sudo apt-get -y install python-dev python-setuptools
sudo apt-get install python-pip python-dev build-essential
sudo pip install py4j
sudo pip install "ipython[notebook]"


# 3- configure Jupyter
# 3-1- create Jupyter configuration file
sudo jupyterhub --generate-config -f /home/datascience/jupyterhub_config.py

# 3-2- create and add SSL certificat, so that your password is not sent unencrypted by your browser
openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -nodes -days 365

# 3-3- create cookie secret
openssl rand -base64 2048 > /home/datascience/cookie_secret
sudo chmod a-srwx /home/datascience/cookie_secret

# 3-4- create auth token
openssl rand -hex 32 > /home/datascience/proxi_auth_token

# 3-5- create auth token
sudo touch /var/log/jupyterhub.log

# 4- configure Jupyter
# 4-1- Edit configuration file
sudo nano /home/datascience/jupyterhub_config.py

# 4-2- Add the content below in configugation file
c = get_config()
# IP and Port
c.JupyterHub.ip = '10.132.0.10' # IP local
c.JupyterHub.port = 443
# Security - SSL
c.JupyterHub.ssl_key = '/home/datascience/key.pem'
c.JupyterHub.ssl_cert = '/home/datascience/cert.pem'
# Security - cookie secret
c.JupyterHub.cookie_secret_file ='/home/datascience/cookie_secret'
c.JupyterHub.db_url = '/home/datascience/jupyterhub.sqlite'
# Security - http token
c.JupyterHub.proxy_auth_token = '/home/datascience/proxi_auth_token'
# put the log file in /var/log
c.JupyterHub.extra_log_file = '/var/log/jupyterhub.log'
# specify users and admin
c.Authenticator.whitelist = {'datascience'}
c.Authenticator.admin_users = {'datascience'}

# 4-3- save jupyterhub_config.py

# 5- configure python 2 kernel

cat <<EOF | sudo tee /usr/local/share/jupyter/kernels/python2.7/kernel.json

{"display_name": "Python 2", 
"language": "python", 
"argv": 
  ["/usr/local/anaconda2/bin/python", 
  "-c", 
  "from ipykernel.kernelapp import main; main()", 
  "-f", 
  "{connection_file}" ]
}
EOF

# 6- test jupyter
# 6-1 Launch Jupyter server
@reboot nohup sudo /usr/bin/python3.5 -m jupyterhub -f /home/datascience/jupyterhub_config.py &

# 7- configure crontab to launch jupyter on start
# Run Jupyter on start up
@reboot nohup sudo /usr/local/python-3.4/bin/python3 -m jupyterhub -f /home/agambo/jupyterhub_config.py &

# Stop server after 00:01:00
1 0 * * * sudo shutdown -h 0

``` 




