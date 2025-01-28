Vishal Lavare (github:- https://github.com/vishalLavare)

To create a nodejs project in jenkins with pulling the code from git into the deployment server, building, testing, and deploying it on the deployment server

we installed nodejs tool and given a name to it we have to check it bcoz we have to write it in the script Dashboard ---- Manage Jebkins ----- Tools ------- Check the name of Nodejs

then we have to install the plugin for doing ssh into the live server Dashboard---Manage Jenkins----Plugin----Available Plugin----search 'SSh agent' and intall it

then we have to create a user for ssh Dashboard ----- Manage Jenkins ---- Credentials ---- Global ---- Add Credentials ----- Kind ----- SSH username with private key ----- Username as Ubuntu --- --- and in private key enter the .pem key of the deployment server ---- create

now create a directory in local (mkdir nodeproject) ----- Create package.json, index.js, test directory and test.js inside test directory ---- ----- push the package.json, index.js and test directory to github

getting ready Deployment server launch a new server and install npm, nodejs, and git in the server ------ create a directory (mynodeapp) and initialise git init

Now create a pipeline give name and description ---- Build Triggers ----> Github hook triggers ---- pipeline syntax ----- in sample step click on checkout from version control --- ---- and paste the git repository url ---- select the branch on which we pushed the code ----- Generate Pipeline Script ----- Copy the script ----------

below is the code of the whole pipeline ------>

pipeline {
    agent any     // it means if we are deploying the code of the available server (jenkins works on master and slave architecture)
    tools {
        nodejs 'mynode'  // Node.js installation configured in Jenkins (this is the tool of nodejs and we enterred the name of it that we are given to it earlier)
    }
    stages {
        stage('Git Clone') {
            steps {
                echo 'Cloning from Git...'
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/vishalLavare/jenkins-nodejs-app.git']])  // this is the script which we have generated through pipeline syntax for version control
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
                echo 'Deploying Node.js project'
                script {
                    sshagent(['8636044c-8504-49d1-aa71-e8ad9df72ba6']) {    // this is the id of the user that we created in []
                        sh '''
                        ssh -o StrictHostKeyChecking=no ubuntu@3.92.192.219 << EOF   // here we given the public ip of the deployment server
                        cd /home/ubuntu/mynodeapp || { echo "Deployment directory does not exist. Exiting..."; exit 1; }
                        git pull https://github.com/vishalLavare/jenkins-nodejs-app.git     // the repository link from where we have to push the code (means our code is  already on that repository)
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
