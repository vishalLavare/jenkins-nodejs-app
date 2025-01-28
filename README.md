# Jenkins Pipeline for Node.js Project Deployment

This guide demonstrates how to create a Jenkins pipeline to build, test, and deploy a Node.js project. The pipeline pulls the code from Git, builds and tests it, and deploys it on a deployment server.

## Prerequisites
1. **Jenkins Setup**: Ensure Jenkins is installed and running.
2. **Node.js Tool Configuration**:
   - Navigate to **Dashboard** -> **Manage Jenkins** -> **Tools** -> **Node.js Installation**.
   - Install Node.js and name it (e.g., `mynode`).
3. **SSH Agent Plugin**:
   - Navigate to **Dashboard** -> **Manage Jenkins** -> **Plugin Manager** -> **Available Plugins**.
   - Search for `SSH Agent` and install it.
4. **SSH Credentials**:
   - Navigate to **Dashboard** -> **Manage Jenkins** -> **Credentials** -> **Global** -> **Add Credentials**.
   - Choose **Kind** as `SSH username with private key`.
   - Provide the username (e.g., `ubuntu`) and the private key (e.g., `.pem` key of the deployment server).

## Setting Up the Project
1. Create a local directory:
   ```bash
   mkdir nodeproject
   ```
2. Add the following files:
   - `package.json`
   - `index.js`
   - `test/test.js`
3. Push the project to a GitHub repository.

## Preparing the Deployment Server
1. Launch a new server.
2. Install the required tools:
   ```bash
   sudo apt update
   sudo apt install -y nodejs npm git
   ```
3. Create a directory and initialize Git:
   ```bash
   mkdir mynodeapp
   cd mynodeapp
   git init
   ```

## Creating the Jenkins Pipeline
1. Navigate to **Dashboard** -> **New Item** -> **Pipeline**.
2. Configure the pipeline:
   - Add a name and description.
   - Under **Build Triggers**, enable **GitHub hook triggers**.
   - Generate the pipeline syntax:
     - Go to **Pipeline Syntax** -> **Checkout from Version Control**.
     - Paste the GitHub repository URL and select the branch.
     - Copy the generated script.
3. Use the following pipeline script:

```groovy
pipeline {
    agent any
    tools {
        nodejs 'mynode'  // Node.js installation configured in Jenkins
    }
    stages {
        stage('Git Clone') {
            steps {
                echo 'Cloning from Git...'
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/vishalLavare/jenkins-nodejs-app.git']])
            }
        }

        stage('Build') {
            steps {
                echo 'Building Node.js project...'
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                echo 'Testing Node.js project...'
                sh './node_modules/mocha/bin/_mocha --exit ./test/test.js'
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying to the server...'
                script {
                    sshagent(['8636044c-8504-49d1-aa71-e8ad9df72ba6']) {  // ID of the SSH credential
                        sh '''
                        ssh -o StrictHostKeyChecking=no ubuntu@3.92.192.219 << EOF
                        cd /home/ubuntu/mynodeapp || { echo "Deployment directory does not exist. Exiting..."; exit 1; }
                        git pull https://github.com/vishalLavare/jenkins-nodejs-app.git
                        npm install
                        sudo npm install -g pm2
                        pm2 restart index.js || pm2 start index.js
                        exit
                        EOF
                        '''
                    }
                }
            }
        }
    }
}
```

## Key Notes
- Replace `mynode` with the name of your Node.js tool in Jenkins.
- Replace `8636044c-8504-49d1-aa71-e8ad9df72ba6` with your SSH credential ID.
- Replace `3.92.192.219` with your deployment server's public IP address.
- Ensure the deployment server directory (`/home/ubuntu/mynodeapp`) exists and is initialized with Git.

## Author
[Vishal Lavare](https://github.com/vishalLavare)

