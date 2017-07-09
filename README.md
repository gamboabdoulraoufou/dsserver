## dsserver

In this post I show how to setput a server for data science project. I will install the following tools: 
- Python (Anaconda distributio)
- Jupyter
- Rstudio
- Keras
- Theano
- Tensoflow
- Xgboost
- googlemaps
- geopy
- H2O


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

# update scikit-learn version
sudo su 
export PATH=$PATH:/usr/local/anaconda2/bin
conda --version
conda install scikit-learn=0.18.1
exit

# check scikit-learn version
/usr/local/anaconda2/bin/python
import sklearn
print('The scikit-learn version is {}.'.format(sklearn.__version__))

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

sudo apt-get install npm nodejs-legacy
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
sudo jupyterhub --generate-config -f /home/agambo/jupyterhub_config.py

# 3-2- create and add SSL certificat, so that your password is not sent unencrypted by your browser
openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -nodes -days 365

# 3-3- create cookie secret
openssl rand -base64 2048 > /home/agambo/cookie_secret
sudo chmod a-srwx /home/agambo/cookie_secret

# 3-4- create auth token
openssl rand -hex 32 > /home/agambo/proxi_auth_token

# 3-5- create auth token
sudo touch /var/log/jupyterhub.log

# 4- configure Jupyter
# 4-1- Edit configuration file
sudo nano /home/agambo/jupyterhub_config.py

# 4-2- Add the content below in configugation file
c = get_config()
# IP and Port
c.JupyterHub.ip = '10.132.0.8' # IP local
c.JupyterHub.port = 9083
# Security - SSL
c.JupyterHub.ssl_key = '/home/agambo/key.pem'
c.JupyterHub.ssl_cert = '/home/agambo/cert.pem'
# Security - cookie secret
c.JupyterHub.cookie_secret_file ='/home/agambo/cookie_secret'
c.JupyterHub.db_url = '/home/agambo/jupyterhub.sqlite'
# Security - http token
c.JupyterHub.proxy_auth_token = '/home/agambo/proxi_auth_token'
# put the log file in /var/log
c.JupyterHub.extra_log_file = '/var/log/jupyterhub.log'
# specify users and admin
c.Authenticator.whitelist = {'abdoul'}
c.Authenticator.admin_users = {'abdoul'}

# 4-3- save jupyterhub_config.py

# 5- configure python 2 kernel

# create file
sudo mkdir -p /usr/local/share/jupyter/kernels/python2.7/

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
sudo /usr/bin/python3.5 -m jupyterhub -f /home/agambo/jupyterhub_config.py 

``` 

> step 3: install Rstudio

```sh  
# 1- install R
# 1-1 setting up apt
sudo sh -c 'echo "deb http://cran.rstudio.com/bin/linux/ubuntu trusty/" >> /etc/apt/sources.list'
gpg --keyserver keyserver.ubuntu.com --recv-key E084DAB9
gpg -a --export E084DAB9 | sudo apt-key add -
sudo apt-get update

# 2- install R
sudo apt-get -y install r-base

# 3- download and install R Studio
sudo apt-get install gdebi-core
wget https://download2.rstudio.org/rstudio-server-1.0.136-amd64.deb
sudo gdebi rstudio-server-1.0.136-amd64.deb

# 4- Configuring RStudio 
# 4-1 create server configuration file
sudo nano /etc/rstudio/rserver.conf

# 4-2 add the following lines and save file
# RStudio port
www-port=8787
# Allow trafic from a specific IP
www-address=0.0.0.0 # world wide

# 4-3- Session configuration
# Create session configuration file
sudo nano /etc/rstudio/rsession.conf
# 4-4 add the following code
# set CRAN 
r-cran-repos=https://mirrors.nics.utk.edu/cran/

# 4-4- User authentication
sudo nano /etc/pam.d/rstudio

#4-5- Add the following code
@include common-auth
@include common-account
@include common-password
@include common-session


# 5- check configuration
sudo rstudio-server verify-installation

# 6- Managing server
# Stop
sudo rstudio-server stop
# Start
sudo rstudio-server start
# Restart
sudo rstudio-server restart
``` 

> step 4: install theano
```sh
# install theano
sudo /usr/local/anaconda2/bin/python -m pip install Theano

# configure theano to use gpu
nano ~/.theanorc

# add the folowwing configuration
[global]
device = gpu
floatX = float32
optimizer_including = cudnn
allow_gc = False
[lib]
cnmem=.95

 ``` 
 
> step 5: install tensorflow
```sh  
# install pip
sudo apt-get install python-pip python-dev

# select the correct binary to install (https://www.tensorflow.org/versions/r0.11/get_started/os_setup)
export TF_BINARY_URL=https://storage.googleapis.com/tensorflow/linux/cpu/tensorflow-0.11.0-cp27-none-linux_x86_64.whl

# upgrage setuptools
sudo /usr/local/anaconda2/bin/python -m pip install --ignore-installed --upgrade pip setuptools

# install TensorFlow
sudo /usr/local/anaconda2/bin/python -m pip install --upgrade $TF_BINARY_URL
``` 

> step 6: install and configure keras
```sh  
# install kera
sudo /usr/local/anaconda2/bin/python -m pip install keras

# change backend  
nano ~/.keras/keras.json

# use the following configuration
{
    "image_data_format": "channels_last",
    "epsilon": 1e-07,
    "floatx": "float32",
    "backend": "theano"
}

``` 

> step 7: install xgboost  
```sh  
# install git
sudo apt-get install git

# download package binary
git clone --recursive https://github.com/dmlc/xgboost

# make
cd xgboost; make -j4

# install xgboost (https://github.com/dmlc/xgboost/blob/master/doc/build.md#building-on-ubuntu-debian)
cd
cd xgboost/python-package
sudo apt-get install python-setuptools
sudo /usr/local/anaconda2/bin/python setup.py install

# export xgboost path
export PYTHONPATH=~/xgboost/python-package

``` 

> step 8: install google maps  
```sh  
# Install googlemaps
sudo /usr/local/anaconda2/bin/python -m pip install -U googlemaps

sudo /usr/local/anaconda2/bin/python -m pip install pyopenssl ndg-httpsclient pyasn1
``` 

> step 9: install geopy  
```sh
sudo /usr/local/anaconda2/bin/python -m pip install geopy
```

> Prepare R for H2O installation  
```sh
# Install dependencies
sudo apt-get install libcurl4-openssl-dev 
sudo apt-get install libxml2-dev

# Install R packages dependencies
## log to R
sudo R

## installa R package
install.packages("RCurl", lib="/usr/local/lib/R/site-library")
install.packages("rjson", lib="/usr/local/lib/R/site-library")
install.packages("statmod", lib="/usr/local/lib/R/site-library")
install.packages("survival", lib="/usr/local/lib/R/site-library")
install.packages("stats", lib="/usr/local/lib/R/site-library") -- bad
install.packages("tools", lib="/usr/local/lib/R/site-library") -- bad
install.packages("utils", lib="/usr/local/lib/R/site-library") -- bad
install.packages("methods", lib="/usr/local/lib/R/site-library") -- bad
``` 

> install and configure H2O for R
```sh  
# install java
sudo apt-get install openjdk-7-jdk

# Downoload package
wget http://h2o-release.s3.amazonaws.com/h2o/rel-vajda/3/h2o-3.10.5.3.zip

# unpack H2O package
unzip h2o-3.10.5.3.zip
cd h2o-3.10.5.3

# Start H2O Server
nohup sudo java -jar h2o.jar &

# Go to http://IP:54321
```

> crontab

```sh  
# 7- configure crontab to launch jupyter on start
export VISUAL=nano

crontab -e

# Run Jupyter on start up
@reboot nohup sudo /usr/bin/python3.5 -m jupyterhub -f /home/agambo/jupyterhub_config.py &

# Run Rstudio on start up
@reboot nohup sudo rstudio-server start &

# Run H2O on start up
@reboot nohup sudo java -jar /home/agambo/h2o-3.10.5.3/h2o.jar &

# Stop server after 23:00:00
1 21 * * * gcloud compute instances stop dsserver --zone europe-west1-d
``` 
