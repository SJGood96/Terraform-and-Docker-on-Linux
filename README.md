# Terraform-and-Docker-on-Linux
This is part 1 of expanding my Linux Homelab using Terrafrom, Docker, and QUEM/KVM


The first part of this series is a focus on Docker and Terraform. I got this idea because I have been interested in learning Terraform as well as learning more about Docker. I knew that I would expand my homelab someday and I figured why not start doing that now to learn some new skills.

I used this repository from Tom Dean.

Before I hopped into using Docker and Terraform I wanted a better understanding of what infrastructure as code is. Which I found here.
What is IaC?

Infrastructure as Code (IaC) allows you to manage the infrastructure through the use of code instead of through the GUI. Management of the infrastructure can be; creating, provisioning, and destroying the infrastructure.

Some Benefits:

    Cost efficient: Keeps cost low by allowing you to destroy it
    Provisioning the same environment
    Reduce Risk

Installing Docker

The first thing that I did was install Docker on Ubuntu. I followed the documentation from Install Docker Desktop due to the documentation saying that I could install Docker Desktop on Ubuntu.

They have instructions on installing a Gnome for a non-Gnome environment but as I already have Gnome I skipped this part.

Set up Docker’s package repository. Use Step 1 (code below)

# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Add the repository to Apt sources:
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

Go to the link here, download the latest DEB package

Install the package with apt as follows:

sudo apt-get update
sudo apt-get install ./docker-desktop-<version>-<arch>.debhere

This is where I got the error: bash: version: No such file or directory. I looked through many different documentation to see what I could do. There was one person who said to put the version and arch in instead of having the code like it is. I did some more research and found the current version and arch. This did not work. I poured myself into research for a day but came up with nothing that worked. One of my connections messaged me about trying after sharing the error code with him:

sudo apt update 

Sudo apt install docker.io docker-compose -y

This did not work either. While looking through documentation I ran across something for Installing Docker Engine on Ubuntu.

Uninstall all the conflicting packages:

for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done

Set up Docker’s apt repository:

# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Add the repository to Apt sources:
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

Install the Docker packages. I used the latest but you can use a specific version if you have one in mind that you want to use.

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

Lastly, test Docker using the “Hello World” image:

sudo docker run hello-world

This should output what the image below shows:
Bash out put from running “hello-world”

The last thing that I did was adding the user to the docker group. I tried:

$ sudo usermod -aG docker `$User`

You will have to log out and back in for this to work. If this works for you then you can skip the next part and go on installing Terraform.

This did not work for me as later on when I was doing the plan function for Terraform I got this error:

I found another document from Docker about Linux post-installation steps for Docker.

Create docker group:

sudo groupadd docker

Add user to docker group

sudo usermod -aG docker $USER

Log out and back in. The activate the changes to group

newgrp docker

Verify that you can run Docker again.

docker run hello-world

Then you should get this:
Bash out put from running “hello-world”
Install Terraform

The next step is to install Terraform. I did this through the Install Terraform documentation.

I made sure to click on Linux and the Ubuntu. You can click on whichever Linux distro you use.

Make sure that everything is up to date and that gnupp, properties-common, and curl packages are installed.

sudo apt-get update && sudo apt-get install -y gnupg software-properties-common

Install the HashiCorp GPG Key

wget -O- https://apt.releases.hashicorp.com/gpg | \
gpg --dearmor | \
sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg

Verify the Key

gpg --no-default-keyring \
--keyring /usr/share/keyrings/hashicorp-archive-keyring.gpg \
--fingerprint

Add the official HashiCorp repo

echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \
https://apt.releases.hashicorp.com $(lsb_release -cs) main" | \
sudo tee /etc/apt/sources.list.d/hashicorp.list

Update

sudo apt update

Then install Terraform from the new repo

sudo apt-get install terraform

Create Infrastructure as Code

Next up is creating a configuration file that provision a nginx container on Docker.

You will have to create a working directory for Terraform. You will need to create a directory called terraform-test then you change the working directory to that directory.

$ mkdir terraform-test
$ cd terraform-test

Next, you can create a file with the Terrafrom code. The touch command creates the main.tf file into the terraform-test directory

$ touch main.tf

The next part is to input the code into the main.tf file. The cat command inputs text into the file.

$ cat > main.tf

Then type:

terraform {
  required_providers {
    docker = {
      source = "kreuzwerker/docker"
    }
  }
}
provider "docker" {
  host = "unix:///var/run/docker.sock"
}
resource "docker_container" "nginx" {
  image = docker_image.nginx.name
  name  = "nginx_container"
  ports {
    internal = 80
    external = 80
  }
}
resource "docker_image" "nginx" {
  name = "nginx:latest"
}

Then hit Ctrl+C to end writing in the file.

The next step is to deploy what you have written.
Deploy

Before deploying the infrastructure as code you need to initialize Terraform.

$ terraform init

This should output something like this:

Validate your Terrafrom Configuration and see what you’re building.

$ terraform plan

Keep doing the plan command until you get no errors.

Now it’s time to provision your resource.

$ terraform apply

Make sure that you type in “yes”. This is one step that I did not do at first and I thought I messed things up but after looking through the repo again and doing it over a few times I realized the human error I made.

You can check these results by looking in your browser at localhost:80 or you can run:

$ docker ps -a

or:

$ curl http://localhost:80

I did not do the last two options but went through the localhost option and saw that it worked.
Destroy Infrastructure as Code

Tearing down the infrastructure is easier than deploying it.

$ terraform destroy

Make sure that you type “yes”. The result should look like this:
Output for the destroy command

You can use three options when verifying that it was destroyed:

First option:

$ curl http://localhost:80

Second option:

$ docker ps -a

The last option is using the Docker image and making sure that it is removed:

$ docker images

Conclusion:

I had a lot of fun working with Terraform and Docker on Ubuntu. Building the Docker infrastructure and making it work. My next phase of this will be combining Terraform and KVM. Hope you can join me on this journey of learning new tools and expanding my homelab.
Resources:

Dean, Tom. “Hands-On-With-Terraform-On-Linux/ at Main · Southsidedean/Hands-On-With-Terraform-On-Linux.” GitHub, github.com/southsidedean/hands-on-with-terraform-on-linux?search=1#hands-on-with-terraform-on-linux.

“Install Docker Desktop on Ubuntu.” Docker Documentation, 20 Oct. 2023, docs.docker.com/desktop/install/ubuntu/.

“Install Docker Engine on Ubuntu.” Docker Documentation, 20 Oct. 2023, docs.docker.com/engine/install/ubuntu/#install-using-the-repository.

“Install Terraform | Terraform | HashiCorp Developer.” Install Terraform | Terraform | HashiCorp Developer, developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli.

“Post-Installation Steps for Linux.” Docker Documentation, 10 May 2022, docs.docker.com/engine/install/linux-postinstall/#manage-docker-as-a-non-root-user.

“What Is Infrastructure as Code with Terraform? | Terraform | HashiCorp Developer.” What Is Infrastructure as Code with Terraform? | Terraform | HashiCorp Developer, developer.hashicorp.com/terraform/tutorials/docker-get-started/infrastructure-as-code.
