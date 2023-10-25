# Jenkins Master-Slave Setup with Docker Compose

This repository provides a Docker Compose setup for running Jenkins in a master-slave configuration using Docker containers. This setup enables scalable and flexible management of Jenkins build agents through Docker.

## Prerequisites

Before you begin, make sure you have the following software installed on your system:

- [Docker](https://www.docker.com/get-started)
- [Docker Compose](https://docs.docker.com/compose/install/)

## Getting Started

1. **Clone the Repository**

   ```bash
   git clone <repository-url>
   cd <repository-directory>
   ```
2. ****Set Environment Variables****

   Create a `.env` file in the project directory with the following environment variables:

   ```bash
    HOST_UID=<your-host-uid>
    HOST_GID=<your-host-gid>
    JENKINS_AGENT_SSH_PUBKEY=<your-ssh-public-key>
    HOST_DOCKER=/var/run/docker.sock  
   ```

   - Generate SSH key pair on the machine where Jenkins agents will run

     ```bash
     ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
     ```
   - Copy public key from *id_rsa.pub* and set `JENKINS_AGENT_SSH_PUBKEY` to value of public key

     ```bash
     JENKINS_AGENT_SSH_PUBKEY=<id_rsa.pub> 
     ```
   - Private key value `id_rsa` will be used later for creating credentials on Jenkins master node ([link](#setting-jenkins-credentials))
   - `HOST_UID` is id of `jenkins` user on host machine

     ```bash
     id -u jenkins
     ```
   - `HOST_GID` is *docker* group id on host machine where `jenkins` user belongs
      ```bash
      getent group docker
      ```
      - **User `jenkins` must belongs to the docker group**

   Replace `<your-host-uid>`, `<your-host-gid>`, and `<your-ssh-public-key>` with your host user ID, group ID, and Jenkins agent SSH public key respectively.
   &nbsp;
3. **Build and Start Jenkins Services**

   ```bas
   docker-compose up -d
   ```
4. **Build and Start Jenkins Services**
   &nbsp;
   Once the services are up and running, access Jenkins in your browser at `http://localhost:50001`. Use the initial setup wizard to configure Jenkins according to your requirements.

## Jenkins Configuration

###### Master Configuration

- **Jenkins URL:** [http://localhost:50001](http://localhost:50001)
- **Volume Mount:** Jenkins data is stored persistently in the `jenkins_data` volume.

###### Docker Slave Configuration

- **SSH Port:** 22
- **Docker Socket:** Mounted from the host (`/var/run/docker.sock`)
- **Volume Mounts:** Jenkins home directory mounted to `/home/jenkins`
- **Exposed Port:** 22 for SSH connections

###### Postman Slave Configuration

- **SSH Port:** 22
- **Volume Mounts:**
  - Postman Newman configuration: `/etc/newman`
  - Jenkins home directory: `/home/jenkins`
- **Exposed Port:** 22 for SSH connections

## Stopping the Services

To stop the Jenkins services, run:

```bash
docker-compose down
```

## Connecting Jenkins Agent to Master Node with SSH Key Pair

#### Setting Jenkins credentials

1. **Login to Jenkins**

   - Open your web browser and go to your Jenkins URL `(usually http://localhost:50001)`.
   - Log in with your Jenkins credentials.
     &nbsp;
2. **Navigate to Manage Jenkins**

   - Click on the *Manage Jenkins* link located in the left sidebar.
     &nbsp;
3. **Credendtials**

   - From the *Manage Jenkins* page, click on *Credendtials* to access the global credendtials settings
     &nbsp;
4. **Add new credential**

   - Select `SSH Username with private key`
   - Set scope to `System (Jenkins and nodes only)`
   - Set `ID` field to be unique credentials identifier in the system
   - Set `Description`
   - Set `Username` to `jenkins`
   - Paste private key in the field below
     - Copy private key `id_rsa` from step created in [link](#getting-started)
       &nbsp;
5. **Configure Jenkins Agent to Use SSH Key:**

   - Create or Configure Jenkins Agent:

     - In Jenkins, navigate to *Manage Jenkins* > *Manage Nodes and Clouds.*
     - Create a new agent or configure an existing agent to use SSH for communication.
     - Set agent name
     - Set *Remote root directory* to be `/home/jenkins/agent`
     - Specify the agent's launch method as *Launch agent via SSH.*
     - Set *Lables* to unique name of agent (this values is used in pipeline to specify on which agent pipeline stage needs to be executed), for example `postman-agent`
       &nbsp;
   - Provide Connection Details:

     - Enter the agent's host name (agent container host name, in this case `jenkins-docker-slave` or `jenkins-postman-slave`) and SSH port (usually 22).
     - Select the saved SSH credentials from the dropdown list.
     - Set *Host Key Verification Strategy* to **Known hosts file Strategy**
     - Save the agent configuration.

&nbsp;
6.  **Test Connection**:

- Before launhing agent, agent needs to be added to **known_hosts** file in master node:
  - Go into master node container bash `docker exec -it jenkins-master bash`
  - If doesn't exists create `.ssh` folder at `/var/jenkins_home`:
    ```bash
    mkdir /var/jenkins_home/.ssh
    ```
  - Create `known_hosts` file:
    ```bash
    touch known_hosts
    ```
  - Navigate to `/var/jenkins_home/.ssh`
  - Add agents to `known_hosts` file:
    ```bash
    ssh-keyscan -H jenkins-docker-slave >> known_hosts
    ssh-keyscan -H jenkins-postman-slave >> known_hosts
    ```
- After saving the agent configuration, Jenkins provides an option to test the connection.
- Click on *Launch agent* or *Test Connection* to ensure Jenkins can connect to the agent using the provided SSH key pair.
- If the test is successful, the agent status will be displayed as online.

Now, the Jenkins agent is securely connected to the master node using the SSH key pair. This setup allows Jenkins to execute jobs and tasks on the agent node via SSH, enabling distributed and parallelized builds and deployments in your CI/CD pipelines.

___

### NOTE

- To add more agents, create appropriate `Dockerfile` file and update `docker-compose.yml` with new services (*agents*). To connect with master node, follow [steps](#connecting-jenkins-agent-to-master-node-with-ssh-key-pair) above.


###### Enable Jenkins to render HTML files

- To enable jenkins to render generated HTML reports, *Content Security Policy* needs to be change. To enable this, go to *Manage Jenkins > Script Console*. In console past the following command and click run:
  ```bash
  System.setProperty("hudson.model.DirectoryBrowserSupport.CSP", "")
  ```