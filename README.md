# Local Server structure  
There are two main components to the local server: the Flask webapp and the Tensorflow Serving module. Both of these components live in Docker containers, and the containers are connect via port 8500 on the TF serving container. And all communication between the camera nodes and the local server are through port 5000 on the Flask app container.

# Install Dependencies for local server 
<!-- **PLEASE USE VIRTUALENV**  
In a virtual env:  
1. In the ```sanus_face_server``` root directory, run ```pip install -r requirements.txt ```  
2. Build dlib python bindings. (Don't use pip if there's CUDA/CuDNN installations on the system) Download [Dlib source](http://dlib.net/files/dlib-19.15.tar.bz2), untar, cd into the directory and run ```python setup.py install``` in your virtualenv. -->


# Install Docker
* Install docker, follow instruction for the OS where tensorflow serving model server will run (macos or linux).  
* The ```docker``` commands listed below might need be ran with root access: ```sudo docker```  

# Build the Dockerfiles and run containers  
## Flask app container:  
* The Dockerfile [Dockerfile.flask-app](https://github.com/sanus-solutions/sanus_face_server/blob/server_dev/Dockerfile.flask-app) builds the container for the Flask app, port 5000 is exposed.  
* Building the dockerfile: ```docker build -t <name_here> -f Dockerfile.flask-app .```  
* Running the docker container: ```docker run -it -p 5000:5000 <name_here>``` Some tags could also be added: ```-d``` for detached container, ```-it``` for interactive session.  

## Tensorflow Serving without GPU support:  
* The Dockerfile [Dockerfile.serving-min](https://github.com/sanus-solutions/sanus_face_server/blob/server_dev/Dockerfile.serving-min) builds the minimum container for tensorflow serving without gpu support.  
* Building the dockerfile: ```docker build -t <name_here> -f Dockerfile.serving-min .```  
* Running the docker container: ``` docker run -it -p 8500:8500 <name_here>```

<!-- 1. There are 2 Dockerfiles. [Dockerfile](https://github.com/sanus-solutions/sanus-face-server/blob/master/Dockerfile) builds the minimal tensorflow serving container without GPU support, and [Dockerfile.devel](https://github.com/sanus-solutions/sanus-face-server/blob/master/Dockerfile.devel)(**Still in development**) builds the tensorflow serving container with GPU support. Note that the GPU support build uses bazel and will eat up all your RAM.  
2. Build the container with: ```docker build --pull -t <your_image_name_here> .```  
    or ```docker build --pull -t <your_image_name_here> -f Dockerfile.devel-gpu .``` 
3. Now run a container with: ```docker run --name <your_container_name_here> -it <your_image_name_here> bash```  
and this will start an interactive bash shell in the container. Now you can run the model server with the following:  
```tensorflow_model_server --port=8500 --model_name=saved_model --model_base_path=/models``` Now the model server should be running in your Docker container. Note that you might have to ```cd ..``` once you're in the container bash. Just make sure you're in a directory that there's a ```\models``` directory.   -->

## Tensorflow Serving with GPU support:  
* The TF serving container with GPU support needs up-to-date NVIDIA driver and the NVIDIA Container Runtime ```nvidia-docker```.  
* Check the list of packages for the gpu using: ```sudo ubuntu-drivers devices``` then use apt-get to install the driver package. For example:  
```sh
sudo ubuntu-drivers devices

== /sys/devices/pci0000:00/0000:00:03.1/0000:23:00.0 ==
vendor   : NVIDIA Corporation  
modalias : pci:v000010DEd00001B80sv00001462sd00003369bc03sc00i00  
driver   : nvidia-384 - distro non-free recommended  
driver   : xserver-xorg-video-nouveau - distro free builtin

sudo apt-get install nvidia-384
```  

* Then install the NVIDIA Runtime ```nvidia-docker```:  
```sh
# If you have nvidia-docker 1.0 installed: we need to remove it and all existing GPU containers
docker volume ls -q -f driver=nvidia-docker | xargs -r -I{} -n1 docker ps -q -a -f volume={} | xargs -r docker rm -f
sudo apt-get purge -y nvidia-docker

# Add the package repositories
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | \
  sudo apt-key add -
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | \
  sudo tee /etc/apt/sources.list.d/nvidia-docker.list
sudo apt-get update

# Install nvidia-docker2 and reload the Docker daemon configuration
sudo apt-get install -y nvidia-docker2
sudo pkill -SIGHUP dockerd

# Test nvidia-smi with the latest official CUDA image
docker run --runtime=nvidia --rm nvidia/cuda nvidia-smi
```
    

    
# Run the local server
In the virtual env that you installed all the dependencies:  
1. In the repo root: ```export FLASK_APP=app.py```
2. Run ```flask run --host=0.0.0.0```
3. You might have to do: ```iptables -I INPUT -p tcp --dport 5000 -j ACCEPT``` to allow port 5000 traffic for Flask.  

# Sending requests to the local server
1. Check the local server's ip address in the router configuration page, and use that address in your request url.  
2. Flask runs on port 5000 by default.  