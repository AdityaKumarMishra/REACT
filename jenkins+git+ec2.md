# Deploying an Amazon EC2 Build Server For ReactJS    with Jenkins and GitHub
![](/image/1.jpeg)


DevOps has been the most discussed subject these days, especially in Continuous Integration/ Continuous Delivery (CI/CD). Therefore, CI/CD has been the main component of the software development cycle, with so many configurations and tools available. We will use Amazon Web Services (AWS) as the cloud platform, Github for the code repository, Jenkins for the Continuous Integration (CI), and AWS CodeDeploy service for the Continuous Delivery (CD).

**Requirements:**

* AWS account.
* Jenkins must be installed on local or on the  AWS EC2 Instance (I will use *Jenkins on EC2).
* Github account.

**Setup an EC2 instance**

1) Login to Amazon AWS and select an EC2 instance
2) Select micro tier and Amazon Linux
3) Create a key pair for the server or choose an existing one
4) Set inbound/outbound security group settings
    * Change HTTP to be open outbound and inbound
    * Change SSH to be open outbound and inbound

5) Assign an elastic IP address to the build server. Use the side menu option on the left hand side Elastic IPs → Allocate new address → Allocate
Apply the Elastic IP address to the new EC2 instance. This will ensure that if the server needs to be restarted we will retain the same IP address. An IP address is assigned on creation and a new one is reassigned on server restarts.

6) SSH into server from Terminal (note: use the newly assigned elastic IP)

``$ ssh -i “build_server.pem” ec2-user@ec2–35.xxx.xxx.xxx.us-west-2.compute.amazonaws.com``

7)  Install Java Runtime and set server path for Java. Jenkins requires the Java to run. From the Terminal:

``$ sudo yum install -y git nginx java-1.8.0-openjdk-devel aws-cli``

 ``$ sudo alternatives — config java``
 
 ``$ export PATH=$PATH:$JAVA_HOME/bin``


8) Install nginx

 ``$ yum install nginx``


9) When Jenkins is installed it runs on port 8080, so a small change needs to be made to proxy Jenkins to port 80.

Change the nginx config (/etc/nginx/nginx.conf):
 ``$ vim /etc/nginx/nginx.conf``

add the following:

`server {`

`listen 80 default;`

`server_name _;`

`location /{`

`proxy_pass http://127.0.0.1:8080;`

`}`
``}``



type esc then :qw! to exit VIM.

10) Restart NGINX

``$ service nginx restart``

11) Install Jenkins. From Terminal command line:

>wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -

>sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
    /etc/apt/sources.list.d/jenkins.list'
    
>sudo apt-get update

>sudo apt-get install jenkins


12) Set Jenkins to start everytime there is a server reboot:

``$ sudo chkconfig — add jenkins``


 13)  Start Jenkins and get secret install key:
 
``$ cat /var/lib/jenkins/secrets/secretfile``

OR, Without Changing into NGNIX CONF File Direct Hit the `IP:8080` to the browser your Jenkins will Start

12) Install the plug-ins for Git and SSH:

  **Manage Jenkins → Manage Plugins → Available tab →GitHub Integration Plugin**
   >Publish Over SSH


13) Create an SSH Key for Jenkins to access GitHub

 From the terminal command:
``$ sudo -su jenkins``
``$ ssh-keygen``

14) Copy key text to put into Jenkins (private key)
``$ cat /home/ec2-user/.ssh/id_rsa``


15) ``Credentials → System → Globals Credentials → Add Credentials → SSH Username with private key → under Private key select ‘Enter directly’``

![](file:///home/aditya/Pictures/2.jpeg)

*Paste the key text in the window and save.*

The Username field can be anything. It should be something to that is identifies the build server. Here I’ve just made it: EC2_build_server. Then OK.

### GitHub Configuration

16) Log into GitHub and enter the project. As mentioned it is possible to use webhooks to signal to Jenkins there has been a code commit and then engage in a a deploy (for continous deployment). In this particlar scenerio, we will create a Deployment key that we will use for when we manually start the build process.

17) Inside settings select ‘Deploy keys’ from the side menu

![](file:///home/aditya/Pictures/3.jpeg)

18) Select ‘Add deploy key’.

You will need to use the PUBLIC SSH key you created. which can be copied from the file id_rsa.pub. To get the key information go back to the Terminal and type the following command in:

``$ cat /home/ec2-user/.ssh/id_rsa.pub``

use this text inside the Key area. The title can be any identifiable name.
(note: Allow write access is not recommended since the build server should only be reading and fetching the repository)

19) Following the NodeJS plugin info:

   [Link](https://wiki.jenkins.io/display/JENKINS/NodeJS+Plugin)
 
 ‘After installing the plugin, go to the global jenkins configuration panel (JENKINS_URL/configure or JENKINS_URL/configureTools if using jenkins 2), and add new NodeJS installations.’
 
 
 20) Back in Jenkins, create a New Item (on the right side main menu)


- Enter any name that is identifiable.
- Select Freestyle project

![](file:///home/aditya/Pictures/4.jpeg)

- Under ‘Source Code Management’ → Select Git

![](file:///home/aditya/Pictures/5.jpeg)

- for Repository URL. use the SSH version of the URL (found inside your GitHub project under Clone/Download)
![](file:///home/aditya/Pictures/6.jpeg)


- Since we will be compiling a React project, make sure to check off the option to ‘Provide Node & npm bin/folder to PATH’ under Build Environment
- 

- Under the dropdown for NodeJS Installation, select the node install.
- 

- Under Build. select Execute Shell. inside the Command window we’ll enter the items that are used on the command line to compile the ReactJS build. These commands will run after the repository is fetched.

![](file:///home/aditya/Pictures/7.jpeg)

**Post-build Actions will contain the SSH transfer after the build. After successful build we’ll configure.**

21) Next let’s test that the code is being fetched and compiled successfully. From the main menu, select the new Jenkins task that was created.
![](file:///home/aditya/Pictures/8.jpeg)


- select ‘Build Now’. A new numbered build will appear in the Build History area below the side menu.

- select the build number

##### Configure SSH

22) Configure the ‘Publish Over SSH Plugin’

- ‘Manage Jenkins’ → ‘Configure System’ → ‘Publish over SSH section’
 
- Inside path to key enter the public key of the Staging server
 
- in the case of EC2 servers, the username will be: ec2-user

- For server, the EC2 instance address: ec2–35–xxx–xxx–xxx.us-west-2.compute.amazonaws.com

- any name that is helpful for identifiable be used, then Add the server
 
23) From the main Jenkins menu. Select the build item again.

* Select Configure
* Select Add post-build action
* Select ‘Send build artifacts over SSH’
* Select the Name from the dropdown

- Inside the source files put the items from the compiled build that you like to move over

24) Run the build!


![](file:///home/aditya/Pictures/9.jpeg)


*Jenkins should now be fetching the GitHub files, compiling, and then finally moving to the staging server*
