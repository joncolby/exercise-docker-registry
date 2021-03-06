Docker Registry
===============

##### Submitter:

Jonathan Colby




####Methodology:

My approach to solve this exercise:

- Use [Packer](https://packer.io/) to create 2 vm types:
  - VirtualBox OVF file
  - Vagrant box
- The vms are created using the packer **virtualbox-iso** builder, based on the ubuntu-14.04.3-server-amd64 ISO
- Docker is installed on the vm ([docs.docker.com](https://docs.docker.com/engine/installation/ubuntulinux/)
- The "official" [Docker Registry project](https://github.com/docker/distribution/blob/master/docs/deploying.md) is used as the registry (registry:2 image).
- Because of simplicity and speed of implementation, configuration/provisioning is done with Bash Shell scripts. Although I could rewrite them in Puppet if requested. 

####Project Structure

	.
	├── LICENSE
	├── README.md
	├── create-docker-registry-ovf.sh (main script)
	├── docker-registry.json  (packer json template)
	├── http
	│   ├── preseed.cfg.orig
	│   └── preseed.cfg.template (template for preseeding)
	├── output-virtualbox-iso/ (OVF output directory)
	├── scripts/ (provisioning shell scripts)
	│   ├── apt_install.sh
	│   ├── cleanup.sh
	│   ├── docker_install.sh
	│   ├── docker_registry_install.sh
	│   ├── docker_service.sh
	│   ├── setup.sh.template
	│   └── vagrant.sh
	└── vagrant (Vagrant BOX output directory)
    	├── Vagrantfile.template
	

####System Requirements

- Internet connectivity
- A recent packer version (developed with 0.8.6, install with Homebrew)
- VirtualBox
- (optional) Vagrant
- git
- terminal program

####Installing

1. Clone the git repo.

2. Locate and open the file `create-docker-registry-ovf.sh`. This file contains variables which you may want to change. For example, `START_VIRTUALBOX_LOCALLY`, `USERNAME`, `PASSWORD`, `INSECURE_REGISTRY`

3. If `START_VIRTUALBOX_LOCALLY` is set to `true`, then the created VirtualBox image will be started on your local VirtualBox.  **Note**, it is left to the user to setup VirtualBox networking. Using Vagrant makes network configuration much easier.


4. Run the script `create-docker-registry-ovf.sh`:
	
    ./create-docker-registry-ovf.sh
    
5. The build process will download the Ubuntu ISO the first time it runs.  This may take a while.  Also, the Docker Registry image will be downloaded from the internet (see methodology above)

6. The generated images are saved to:  `output-virtualbox-iso` and `vagrant`
	
####TLS - Extra Configuration

A Private Docker Registry should have TLS certificates installed and configured.

For testing, it is possible to run the registry in either INSECURE mode or with a self-signed-certificate

The flag `INSECURE_REGISTRY` can be used to set the registry VM to un insecurely (http).  **However, the docker client configuration must in both cases be updated**

See here:

[https://github.com/docker/distribution/blob/master/docs/insecure.md](https://github.com/docker/distribution/blob/master/docs/insecure.md)

#####Running Vagrant

The `create-docker-registry-ovf.sh` will attempt to add the vagrant image box if vagrant was detected.  

The manual steps are:

	cd vagrant
	vagrant box add --name weltn24-docker-registry weltn24-docker-registry_virtualbox.box
	vagrant init
	
Edit Vagrantfile to configure network settings: 
	
	config.vm.box = "weltn24-docker-registry"
	config.vm.network "private_network", ip: "192.168.33.33" (example)	

Login:

	vagrant ssh
	
Docker Registry should be running:

	root@weltn24-docker-registry:~# docker ps
	CONTAINER ID        IMAGE               COMMAND             	CREATED             STATUS              PORTS    NAMES
	2839f2bd5992        registry:latest     "docker-registry"   43 	minutes ago      Up 2 minutes        0.0.0.0:5000->5000/tcp   	registry
	
The registry is accessible at:

	http(s)://192.168.33.33:5000/
	(or the IP configured for Vagrant)
	

####Running VirtualBox




####Testing the Docker Registry

pull an image locally:

	docker pull ubuntu

tag the image

	docker tag ubuntu <ip-address>:5000/weltn24
	
push the image

	docker push <ip-address>:5000/weltn24

pull it again

	docker pull <ip-address>:5000/weltn24
	

####Tech Notes:

 - docker container is started/restarted with the docker restart policy feature (another option would be supervisord or even upstart)
 - The registry runs in TLS mode with self-signed certificates
 - The registry storage volume is a local mount on the vm docker-host
 - The registry runs without access restrictions (no authorization)
 
 




