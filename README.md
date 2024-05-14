#### #### #### ####   Reg_App_CI-CD_Pipeline
#### This repo has a simple register application hosted on a Tomcat Server . Follow the below guidelines to create a CI/CD job out of it with tools such as EKS,Ansible,Jenkins,Docker.
#### Basic understanding of all these tools are necessary to complete the setup.
#### Fork this repo to your GITHUB - https://github.com/Ashfaque-9x/registration-app
#### Launch 3 device in EC2 with t2.medium, Jenkins server, Ansible server , EKS bootstrap server

      #### optional - change hostname on those EC2's and reboot #### It is done to make more clarity.
      sudo su
      hostname Ansible-Server
      sudo vi /etc/hostname ####  remove everything and add Ansible ; Jenkins ;EKS on three servers
      init 6 #### command to reboot the device

      #### #### Phase-1 - Cluster Creation
      @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
      #### perform cluster creation on EKS server as it takes 20 mins to create
      #### bootstrapping eks cluster
      ----on Amazon linux  ----
      #### create and attach an IAM role with these permissions - Admin access;amazonec2fullaccess;awscloudformationfullaccess;iamfullaccess to this EC2 server
      #### all three should be latest version
      sudo su
      #### aws cli installation and env variable addition
      sudo yum remove awscli -y
      curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
      sudo yum install unzip -y
      unzip awscliv2.zip
      sudo ./aws/install
      aws --version #### no problem if output is displayed ;if not then
      /usr/local/bin/aws --version #### this should get us the  display 
      cd ~
      ls -alt
      vi .bash_profile #### edit $PATH and make -   PATH=$PATH:$HOME/bin:/usr/local/bin
      ####  vi .bashrc in case $PATH does not exist in .bash_profile ;changes based on the OS type
      source .bash_profile #### save it
      echo $PATH
      aws --version #### it should come
      aws configure ####  enter access key ;token ;ap-south-1 region and json
      #### kubectl installation
      curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
      sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
      kubectl version #### if ouput is displayed problem ;if not  then need to add it in env variable (here /usr/local/bin is arlready added for awscli so not needed)
      #### eksctl instllation
      vi script.sh
        ---
        ARCH=amd64
        PLATFORM=$(uname -s)_$ARCH
        curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"
        curl -sL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_checksums.txt" | grep $PLATFORM | sha256sum --check
        tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz
        sudo mv /tmp/eksctl /usr/local/bin
        ---
      chmod +x script.sh
      sh script.sh
      eksctl version #### if ouput is displayed problem ;if not  then need to add it in env variable (here /usr/local/bin is arlready added for awscli so not needed)
      #### all three should work from any path 
      kubectl version
      aws --version
      eksctl version
      #### create cluster and get nodes
      eksctl create cluster --name cluster-linux --region ap-south-1 --node-type t2.small
      kubectl get nodes
      #### now check k get pods and k get nodes
      ---- on ubuntu -----
      #### same thing but replace yum with apt for aws cli installation thats all

      #### #### Phase-2 - Jenkins configuration
      @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
      #### Jenkins, java11, maven, git Installation on EC2
      sudo su
      yum update -y
      wget -O /etc/yum.repos.d/jenkins.repo \
          https://pkg.jenkins.io/redhat-stable/jenkins.repo
      sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
      sudo yum upgrade
      amazon-linux-extras install epel -y 
      sudo amazon-linux-extras install java-openjdk11 -y
      yum install java-11-amazon-corretto -y
      sudo yum install jenkins -y
      sudo systemctl enable jenkins
      sudo systemctl start jenkins
      #### Verification
      java -version
      javac -version
      systemctl status jenkins
      #### Maven Installation and Configuration on Jenkins-Server
      #### basically we are adding the maven env variable to path  #### https://maven.apache.org/download.cgi
      #### Copy the download link from https://maven.apache.org/download.cgi for Binary tar.gz archive
      #### which is https://dlcdn.apache.org/maven/maven-3/3.9.6/binaries/apache-maven-3.9.6-bin.tar.gz for now
      sudo su  & cd /root
      cd /opt
      wget https://dlcdn.apache.org/maven/maven-3/3.9.6/binaries/apache-maven-3.9.6-bin.tar.gz
      tar -xzvf apache-maven-3.9.6-bin.tar.gz
      mv apache-maven-3.9.6 maven
      cd /root
      ls -alt      #### //It will show the hidden files also
      find / -name java-11* #### find the java11 path #### this is later needed for jenkins setup as well
      vi .bash_profile
      #### //enter below lines below the 2nd fi
      M2_HOME=/opt/maven
      M2=/opt/maven/bin
      JAVA_HOME=/usr/lib/jvm/java-11-openjdk-11.0.22.0.7-1.amzn2.0.1.x86_64
      PATH=$PATH:$HOME/bin:$JAVA_HOME:$M2_HOME:$M2
      echo $PATH
      source .bash_profile
      echo $PATH
      mvn -v
      #### installing Git in amazon linux
      yum update -y
      yum install git -y
      %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
      In jenkins management console
        1. Install 'maven integration' plugin
        2. Add java11 and maven configuration (Manage Jenkins-->tools) #### find / -name java-11* #### refer snips Java11 and maven
        3. Install Github plugin and disable Github Branch Source Plugin #### refer snip github
        4. Build CI Job with git URL , main branch,poll scm,  clean install options and verify output in /webapp/target - a new war should be generated #### refer snip from CI-job1 to cijob3 #### https://github.com/Velkaandhans/registration-app.git
      %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

      #### #### Phase-3 - Ansbile Configuration
      @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
      #### Ansible,Docker,Ansible - useraccess,Ansible - inventory file Setup on Ansible Server
      #### Ansible Installation
      sudo su 
      amazon-linux-extras install ansible2 -y
      ansible --version
      #### adminuser creation for ansible
      useradd ansadmin
      passwd ansadmin
      visudo 
      ansadmin ALL=(ALL) NOPASSWD: ALL #### //add this in sudo file.
      cd /etc/ssh
      vi sshd_config #### passwordauthentication yes
      service sshd reload
      sudo su ansadmin
      ssh-keygen
      ssh-copy-id local-host-ip  #### we do this to allow ansible to work on its own server
      public key is at /home/ansadmin/.ssh/id_rsa.pub
      #### creating workdir for the project 
      sudo mkdir /opt/Docker
      sudo chown ansadmin:ansadmin /opt/Docker
      #### Installing docker #### refer this link for instructions -  https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/install-docker.html
      sudo su
      sudo amazon-linux-extras install docker -y
      sudo service docker start
      sudo usermod -a -G docker ansadmin #### restart or signout signin again
      docker ps #### should work for ansadmin 
      #### find the private ip's of Ansible server and EKS server and add them to the Ansible invenotry
      ifconfig 
      sudo vi /etc/ansible/hosts
      [ansible]
      172.31.13.138 #### Private-ip-of-linux-device
      [kubernetes]
      172.31.12.245
      public-ip #### if the server is in different region/vpc
      ssh-copy-id local-host-ip
      #### Creating docker file under /opt/Docker
      cd /opt/Docker
      vi Dockerfile
      FROM tomcat:latest
      RUN cp -R /usr/local/tomcat/webapps.dist/* /usr/local/tomcat/webapps
      COPY ./*.war /usr/local/tomcat/webapps #### this war file will be created from jenkins server and copied to this location as we mentioned in the build configurations of the job
      #### create and test the push of docker images to the repo
      cd /opt/Docker
      docker images
      docker login
      vi regapp.yaml
      - hosts: ansible
        tasks:
        - name: create docker image
          command: docker build -t regapp:latest .
          args:
          chdir: /opt/Docker
        - name: create tag to push image onto dockerhub
          command: docker tag regapp:latest velkaandhan/regapp:latest

        - name: push docker image
          command: docker push velkaandhan/regapp:latest
      ansible-playbook regapp.yaml --check 
      ansible-playbook regapp.yaml #### war file should be there to work correctly
      #### verify the image from docker hub
      %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
      #### In jenkins management console configure Ansible SSH in jenkins
          1. Install a plugin called 'publish over ssh' 
          2. Configure Ansible server in jenkins and test configurations (Manage Jenkins --> system --> SSH Server) #### refer image Ansible
          #### now modify the build in jenkins such that the created war file is sent to the ansible server and a Docker image is created out of it from tomcat image
          3. In post build steps of the ci job -- send build artifacts over ssh #### refer image CI job Mod
          #### Source files:webapp/target/*.war       Remove prefix:webapp/target        Remote directory://opt//Docker (Note the Double slash in remote dir )
          #### Exec Command: ansible-playbook /opt/Docker/regapp.yaml 
          4. verify the build by running it - war file should be there in /opt/Docker and a new Docker image must be there in the Docker hub
      %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
      #### #### Phase-4 - Complete CICD Job
      @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
      #### Create K8 deployment and svc for the app to launch in the EKS cluster
      cd /root #### these files must be kept in root
      ---regapp-deployment.yaml

          apiVersion: apps/v1
          kind: Deployment
          metadata:
            creationTimestamp: null
            labels:
              app: dep-regapp
            name: dep-regapp
          spec:
            replicas: 2
            selector:
              matchLabels:
                app: dep-regapp
            strategy: {}
            template:
              metadata:
                creationTimestamp: null
                labels:
                  app: dep-regapp
              spec:
                containers:
                - image: velkaandhan/regapp
                  name: regapp
                  ports:
                  - containerPort: 8080
                  resources: {}
          status: {}
      ---regapp-service.yaml

          apiVersion: v1
          kind: Service
          metadata:
            creationTimestamp: null
            labels:
              app: dep-regapp
            name: dep-regapp
          spec:
            ports:
            - port: 8080
              protocol: TCP
              targetPort: 8080
            selector:
              app: dep-regapp
            type: LoadBalancer
          status:
            loadBalancer: {}
      ---
      @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
      #### for integrating bootstrap server with Ansible
      #### perform these from bootstrap server
      #### enabling password authentication to server
      sudo su
      vi /etc/ssh/sshd_config
      passwordauthentication yes
      service sshd reload
      #### changing root passwd for ssh login via ansible server
      passwd root #### change the passwd
      #### from ansible server perform a root ssh login to bootstrap server 
      ssh-copy-id root@172.31.12.245 #### will log into boootstrap server from ansible server #### private ip of boostrap server
      #### verification
      ssh root@172.31.12.245
      @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
      #### creating ansible playbook to run deployment and svc yaml files
      cd /opt/Docker
      vi kube_deploy.yaml
      ---
      - hosts: kubernetes
        user: root

        tasks:
          - name: deploy regapp on kubernetes
            command: kubectl apply -f regapp-deployment.yaml

          - name: create service for regapp
            command: kubectl apply -f regapp-service.yaml

          - name: update deployment with new pods if image updated in docker hub
            command: kubectl rollout restart deployment.apps/dep-regapp
      ansible-playbook /opt/Docker/kube_deploy.yaml --check
      %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
      #### In Jenkins Management Console 
          1. create a new freestyle project  'CD Job' only with postbuild actions and choose 'Send build artifacts over SSH' 
              #### ansible-playbook /opt/Docker/kube_deploy.yaml #### Refer Snip - CD Job
          2. Modify the first job such that the post build action contains another job - This CD Job #### refer image - Final CI Config

      #### commit changes to the git hub repo and verfify the following
        1. A new war file should be generated in /opt/Docker in ansbile server
        2. docker images should be built, tagged and uploaded to docker hub
        3. CD should run the kube_deploy.yaml and load balncer url can be verified from EKS server
            #### copy the dns to the browser and add port 8080 #### will take some time 
            http://a7ba1e968fb764ed089553cb4804fe1f-6084869.ap-south-1.elb.amazonaws.com:8080
            #### to get into the app
            http://a7ba1e968fb764ed089553cb4804fe1f-6084869.ap-south-1.elb.amazonaws.com:8080/webapp/
      %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

      @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
      #### env Clean up
      eksctl delete cluster cluster-linux --region ap-south-1 #### or 
      eksctl delete cluster --region=ap-south-1 --name=cluster-linux
#### delete the vms from AWS
