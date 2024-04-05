# Homelab

## Getting started
My homelab is a self-hosted laboratory running on my local network. It’s powered by Docker containers, allowing me to experiment, learn, and deploy services efficiently. 🏠🚀🐳

## Prerequisites
To set up and utilize your homelab effectively, ensure you have the following prerequisites:

1. **Registered Domain for SSL Certificates**:
   - Obtain a domain name (e.g., myhomelab.xyz) and register it with a domain registrar. This domain will be used for SSL certificates to secure your services.
   
   - You can use services like **Let's Encrypt** to issue free SSL certificates for your domain.

2. **Install Docker and Docker Compose**:
   - Install **Docker** on your local machine or server. Docker allows you to create, deploy, and manage containers.

   - Additionally, install **Docker Compose**, which simplifies managing multi-container applications.

3. **AWS Account**:
   - Sign up for an **Amazon Web Services (AWS)** account if you don't have one already.

   - AWS provides various services that can complement your homelab for DNS registration and DNS management.

4. **Install AWS CLI**:
    - Install the awscli on your machine where you want to run the homelab.

    - [Install AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)


Once you've met these prerequisites, your homelab will be ready for exploration and experimentation! 🏠🚀

## Setup

1. **Clone the repository**
    - ``` https://github.com/mrcpngn/homelab.git ```.

2. **Create Record sets on Route 53 using Cloudformation template**
    - Go to the ```aws/``` directory and enter the following commands:
        ```
        STACK_NAME='NginxProxyRoleForHomelab'
        CF_NAME='NginxProxyRole.yml'

        aws cloudformation validate-template \
            --template-body file://$CF_NAME

        aws cloudformation deploy \
            --template-file $CF_NAME \
            --stack-name $STACK_NAME \
            --capabilities CAPABILITY_NAMED_IAM \
            --parameter-overrides \
            DomainName=<Registered-Domain> \
            NginxProxyIPAddress=<Nginx-IP-Address> \
            NginxProxyTTL=<TTL-in-seconds>
        ```

3. **Generate the Nginx IAM Role Credentials**
    - Execute the command to generate the credentials.

        ```
        ROLE_ARN=$(aws iam list-roles --query 'Roles[?starts_with(RoleName, `'$STACK_NAME'`) == `true`].Arn' --output text)
        
        aws sts assume-role --role-arn $ROLE_ARN \
            --role-session-name $STACK_NAME-Session \
            --query 'Credentials'
        ```
    - Make sure to save the credentials into a textpad, you will use this later.
    
4. **Deploy the containers using docker-compose**
    - Change directory and go to the ```infra/``` folder.

    - Run the command ```docker-compose up -d``` to deploy the containers.

5. **Add the SSL Certificates using the Nginx Proxy Manager**
    - Open your browser and enter the ```http://Nginx-IP-Address:81``` where Nginx Proxy manager is hosted.

    - Login using the default credentials ```admin@example.com``` and ```changeme```.

        ![Screenshot](screenshots/nginx-home.png)

    - Nginx Proxy Manager's WebUI, go to SSL Certificates, and click "Add SSL Certificate".

    - Input you domain name (e.g., homelab.xyz), as well as the wildcard record for all subdomains (e.g., *.homelab.xyz).

    - Enable the "Use a DNS Challange", on the DNS provider choose ```Route 53 (Amazon)```.

    - Paste the Nginx IAM Role credentials the "Credentials File Contents" textbox.
        ```
        [default]
        aws_access_key_id=AKIAIOSFODNN7EXAMPLE
        aws_secret_access_key=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
        aws_session_token=QWERTYUASDFGHLEXAMPLESESSIONTOKEN
        ```
    - Click "Save" and wait for a few minutes to propagate the changes.

    - If you did everything correctly, you should now see your generated certificate:

        ![Screenshot](screenshots/nginx-ssl.png)

6. **Adding a proxy entry**
    - Let's add our first proxy entry for the Nginx Proxy Manager itself. This will let us access it by going to ```https://nginx.homelab.xyz```.

    - In the top menu bar, click on Hosts > Proxy Hosts, and then click on "Add a Proxy Host". 
    - I will use the following parameters:
        - Domain Names: nginx.homelab.xyz
        - Scheme: http
        - Forward Hostname / IP: nginxproxymanager (more on that below)
        - Forward Port :81

        ![Screenshot](screenshots/nginx-proxy-host-add.png)
    
    - On the SSL, select the newly add SSL ```homelab.xyz``` and ```*.homelab.xyz```.

        ![Screenshot](screenshots/nginx-proxy-host-ssl1.png)

    - Enable the ```Force SSL``` and ```HTTP/2 Support``` then click "Save".

        ![Screenshot](screenshots/nginx-proxy-host-ssl2.png)

    - Repeat for every application that you'd like to access with your new domain. 🌟

## Acknowledgment
A heartfelt thank you to the https://github.com/notthebee for inspiring me to create this project. Your contributions and creativity have fueled my journey! 🙌🌟