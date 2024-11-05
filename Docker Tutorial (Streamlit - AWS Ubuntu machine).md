# **Streamlit Docker Application Setup**

# 1. Requirements

- AWS account
- A terminal or SSH client for SSH access

Make sure you have the files needed for the project:

- app.py
- requirements.txt
- other files that is used in the app (model.h5, tokenizer….)

---

# 2. Go to AWS EC2 services & create an instance

[us-east-1.console.aws.amazon.com](https://us-east-1.console.aws.amazon.com/console/home?region=us-east-1)

Create an instance:

![image](https://github.com/user-attachments/assets/2e3e5087-554b-4120-af0b-ef4b29147f76)


- Click “launch instance” and complete the sections:
- Give your instance a name
- Choose a machine (e.g. Windows, MAC, Ubuntu…)
- Create a new key pair for ssh connection (name it as you wish, RSA, .pem file)
- .pem file will automatically be downloaded
- Network settings
    - SSH(automatically selected)
    - Add streamlit port: Click edit → Add security group rule → change source type to anywhere and enter the port range for streamlit (8501)
    
    ![image 1](https://github.com/user-attachments/assets/b0b22179-e42f-41a4-8fb4-38a570d4d651)

    
- Configure storage: up to 30 GiB is free, we will make it 20
  
    ![image 2](https://github.com/user-attachments/assets/777cd3bc-b4f2-4ccd-8faa-6832a05d6cef)

    
- Lastly, Click “Launch Instance”

In summary:

Log into the AWS management console and create an EC2 instance with the following specifications:

- **AMI**: Ubuntu 24.04
- **Instance Type**: t2.micro
- **Storage (Volume)**: 20 GB
- **Security Group**:
    - TCP 8501 (for Streamlit application access)
    - TCP 22 (for SSH access)

---

# 3. **Connect to the EC2 Instance via SSH**

Before starting, make sure Remote - SSH is installed in your VSCode:

![image 3](https://github.com/user-attachments/assets/fd0c02b3-fac4-4fb4-9601-39815623ef53)


1. Go to VSCode

![image 4](https://github.com/user-attachments/assets/ed36f111-9433-4656-86a3-e39400b903b6)


1. click the blue icon in the bottom left corner (Open a remote window)
2. Connect to Host…
3. Configure SSH Hosts…

![image 5](https://github.com/user-attachments/assets/409f4b35-6260-4a62-a8b9-76174882482d)


1. Click ..ssh\config option

![image 6](https://github.com/user-attachments/assets/6c4a86ca-cc0e-4f8b-8cc3-634a030512c0)


1. You should see a config file opened
2. Now paste these into that config file (you do not have to delete anything, just add):

```markdown
Host Myawsdocker
  HostName <EC2_PUBLIC_IP>
  IdentityFile ~/Downloads/docker.pem
  User ubuntu
  ServerAliveInterval 60
```

- Myawsdocker → could be any name you give
- Hostname `<EC2_PUBLIC_IP>` → go back to AWS and click your instance, you will see the Public IPv4 address there, copy and paste it here
- IdentityFile  → path to the .pem file that has been downloaded from AWS
- User → Created machine from AWS

### Alternatively, you can connect to the EC2 instance by pasting this in your VSCode terminal:

```markdown
ssh -i "your-key.pem" ubuntu@<EC2_PUBLIC_IP>
```

1. Again, click open a remote window from the bottom corner → Connect to Host →  click the host name you previously created (in our case it is “Myawsdocker”)
2. A new window will open and you should see the host name in the bottom left corner

# 4. Drag your project files to the space and open terminal

if your files are zipped, simply type these commands in your terminal:

```markdown
sudo apt install unzip
unzip Your_project_file.zip
```

# 5. Installing Docker

Use the following commands to install Docker:

```bash
curl -fsSL <https://get.docker.com> -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER
newgrp docker
docker version
```

These steps install the latest version of Docker and add the current user to the `docker` group, allowing Docker commands to run without root privileges. The last 3 commands are to prevent permission errors.

# 6. Removing Docker (Optional)

If you want to remove Docker, you can use the following commands:

```bash
sudo apt-get purge docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-ce-rootless-extras
```

# 7. Creating a Dockerfile

Create a Dockerfile for the image we will build.

```bash
touch Dockerfile
vim Dockerfile
```

Then, add the following content to the file (Python docker file):

```
# Use Python 3.10 as the base image
FROM python:3.11.4-slim

# Set the working directory
WORKDIR /app

# Copy the requirements.txt file to install dependencies
COPY requirements.txt .

# Install the necessary packages
RUN pip install --no-cache-dir -r requirements.txt

# Copy application files to the working directory
COPY . .

# Expose the port for external access
EXPOSE 8501

# Command to start the Streamlit application
CMD ["streamlit", "run", "Your_App.py", "--server.port=8501", "--server.address=0.0.0.0"]
```

Notice you have to replace "Your_App.py" with your original ‘app.py’ file.

# 8. Build the Docker Image

IMPORTANT: Build the Docker image for your application in the directory where the Dockerfile and application files are located:

Simply change directory by: `cd [path to the directory you want to change to]`

```bash
docker build -t streamlit-app .
```

This command reads your `Dockerfile` and creates a Docker image named `streamlit-app`.

# 9 . List Docker Images

To list the Docker images:

```bash
docker images
```

This command shows all available Docker images on the system.

# 10. Deploy the Application in a Docker Container

To run your application using the created image:

```bash
docker run -p 8501:8501 streamlit-app
```

This command starts the Streamlit application and makes it accessible on port 8501.

# 11. Run in Detached Mode

To run the application in the background (detached mode), add the `-d` flag:

```bash
docker run -d -p 8501:8501 streamlit-app
```

This command runs the application in the background, freeing up the terminal.

# 12. List Running Containers

To list all currently running Docker containers:

```bash
docker ps
```

This command shows only the running containers.

# 13. Stop a Running Container

To stop a specific container:

```bash
docker stop <container_id>
```

Replace `<container_id>` with the ID from the `docker ps` command.

# 14. List All Containers

To see all containers (including stopped ones):

```bash
docker ps -a
```

This command lists all Docker containers.

# 15. Remove a Specific Container

Before removing a container, you need to stop it. After stopping, you can remove it with:

```bash
docker rm <container_id>
docker rm $(docker ps -aq) # Removes all stopped containers.
```

Replace `<container_id>` with the ID of the container you wish to delete.

# Notes:

After completing the setup steps, you can access your Streamlit application at `http://<EC2_PUBLIC_IP>:8501`.
