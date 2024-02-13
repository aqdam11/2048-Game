# 2048-Game
Steps:-

Step 1: Create an ubuntu VM

Step 2: Install Jenkins, Docker and Trivy. Create a Sonarqube Container using Docker.
		2.1: Jenkins:
			 sudo apt upgrade -y
			 sudo apt install -y openjdk-11-jdk. //install the latest java version
			 wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
			 sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
			 sudo apt update
			 curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee
			 /usr/share/keyrings/jenkins-keyring.asc > /dev/null $ echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]
			 https://pkg.jenkins.io/debian binary/ | sudo tee
			 /etc/apt/sources.list.d/jenkins.list > /dev/null
			 add this two command if you are getting error
			 sudo apt install -y jenkins
			 sudo systemctl start jenkins
			 sudo systemctl enable jenkins
			 sudo cat /var/lib/jenkins/secrets/initialAdminPassword //use this command to get the admin password.
			 add inbound rule of 8080 (jenkins) for your vm
			 Now login http://ip:8080
			 use password you got
			 install jenkins

		2.2: Docker:
			 sudo apt-get update
			 sudo apt-get install docker.io -y
			 sudo usermod -aG docker $USER  (whoami to check user)
			 newgrp docker
			 sudo chmod 777 /var/run/docker.sock

		2.3: Sonarqube:
			 First allow port 9000
			 docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
			 docker ps to check if its running
			 NOTE: If you stop your VM run below command to re run without losing data.
			 docker run -d -p 9000:9000 -v sonarqube_data:/opt/sonarqube/data -v sonarqube_logs:/opt/sonarqube/logs sonarqube:lts-community. 
			 usename: admmin
			 password: admin

		2.4: Trivy:
			 sudo apt-get install wget apt-transport-https gnupg lsb-release -y
			 wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
			 echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
			 sudo apt-get update
			 sudo apt-get install trivy -y

Step 3 — Install Plugins like JDK, Sonarqube Scanner, Nodejs, and OWASP Dependency Check
		3.1: Go to manage jenkins -> Plugins -> Available Plugins. and install below plugins:
			 Eclipse Temurin Installer
			 Sonarqube Scanner
			 NodeJS Plugin
		3.2: Configuring Java and Nodejs
			 Go to manage jenkins -> tools -> install jdk17 (version jdk-17.0.8+1) //auto install from adoptium.net
			 Go to manage jenkins -> tools -> install nodejs (version nodejs 16.2.0) //auto install from nodejs.org
		3.3: Create a job as 2048 Game, select pipeline and click OK

Step 4 — Create a Pipeline Project in Jenkins using a Declarative Pipeline
		4.1: Configure sonar server in Manage Jenkins
			 ip:9000
			 Administration → Security → Users → Click on Tokens and Update Token → Give it a name → and click on Generate Token , Create a token with a name and generate.
			 Copy Token
		4.2: Goto Jenkins Dashboard → Manage Jenkins → Credentials → Add Secret Text.
			 Give id name
		4.3: go to Dashboard → Manage Jenkins → System
			 Give a name, url will be sonar server i.e ip:9000, authentication will be id name of your creds from last step

		The Configure System option is used in Jenkins to configure different server
		Global Tool Configuration is used to configure different tools that we install using Plugins
		We will install a sonar scanner in the tools.

		4.4: Go to manage jenkins -> tools -> install sonarqube scanner (version 5.0.1.3006) //auto install from adoptium.net // auto install from Maven Central
		4.5: Go to Sonarqube dashboard
			 Administration → Configuration →Webhooks
			 Cick Create
			 Name: jenkins, url: ip:8080/sonarqube-webhooks/
			 Create a pipeline script // Complete script attached as script
			 Click Build

Step 5 — Install OWASP Dependency Check Plugins
		5.1: Go to Dashboard → Manage Jenkins → Plugins → OWASP Dependency-Check.
		5.2: Go to manage jenkins -> tools -> install Dependency Check(version dependency-check 6.5.1) // auto install from github.com


Step 6 — Docker Image Build and Push
		6.1: Goto Dashboard → Manage Plugins → Available plugins → Search for Docker and install these plugins
			 Docker, Docker Commons, Docker pipeline, Docker API, docker-build-step
		6.2: go to manage jenkins -> tools -> install Docker installations(version latest) //auto install from docker.com
		6.3: Add Docker Hub Username and Password under Global Credentials
			 Kind: username with password, scope: Global, username: your docker usernmae: password: docekr password., id: docker: dedcription: docker
		6.4: After build now, you will see a new image is created in your docker hub

Step 7 — Deploy the image using Docker
		7.1: Now Run the container to see if the game coming up or not by adding below stage 
			 Add the Deploy to container stage in script and run it
		7.2: ip:3000 in your browser to check

Step 8 — Kubernetes master and slave setup on Ubuntu (20.04)
		8.1: Create two ubuntu VM, one for master and one for slave
		8.2: Install kubectl in your jenkins vm
			 sudo apt update
			 sudo apt install curl -y
			 curl -LO https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl
			 sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
			 kubectl version --client
		8.3: Master node VM
			 sudo hostnamectl set-hostname K8s-Master
		8.4: Worker node VM
			 sudo hostnamectl set-hostname K8s-Worker
		8.5: For Both Master and Worker node run below commands to setup kubernetes setup
			 sudo apt-get update 
			 sudo apt-get install -y docker.io
			 sudo usermod –aG docker Ubuntu
			 newgrp docker
			 sudo chmod 777 /var/run/docker.sock
			 sudo curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

			 sudo tee /etc/apt/sources.list.d/kubernetes.list <<EOF
			 deb https://apt.kubernetes.io/ kubernetes-xenial main     # 3lines same command EOF

			 sudo apt-get update
			 sudo apt-get install -y kubelet kubeadm kubectl
			 sudo snap install kube-apiserver
		8.6: For Master node:
			 sudo kubeadm init --pod-network-cidr=10.244.0.0/16
			 # in case your in root exit from it and run below commands
			 mkdir -p $HOME/.kube
			 sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
			 sudo chown $(id -u):$(id -g) $HOME/.kube/config
			 kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
		8.7: For Worker node:
			 sudo kubeadm join <master-node-ip>:<master-node-port> --token <token> --discovery-token-ca-cert-hash <hash>
		8.8: Copy the config file to Jenkins master or the local file manager and save it
			 copy it and save it in any folder save it as secret-file.txt
			 Note: create a secret-file.txt in your file explorer save the config in it and use this at the Kuberenetes credential section.

		8.9: Install Kubernetes plugins in Jenkins:
			 Kubernetes Credentials, Kubernetes Client API, Kubernetes, Kubernetes CLI

		8.10: go to manage Jenkins → manage credentials → Click on Jenkins global → add credentials
			  Kind: Secret file, Scope: Global, File: Upload the file which you saved earlier in your system, id: k8s, desc: k8s
			  add the last stage in your script //deploy to kubernetes
		8.11: In the kubernetes cluster, give this command
			  kubectl get all 
			  -> you will find the port under service/petshop
			  kubectl get svc 

Step 9 — Access the Game on Browser.
		9.1: ip of slave:service port





