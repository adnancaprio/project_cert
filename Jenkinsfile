pipeline {
    agent any

    environment {
        // Use the Jenkins SSH credentials ID for your EC2 key
        SSH_KEY = 'WebAppPaC'
        EC2_USER = 'ubuntu'
        EC2_HOST = '3.131.141.176'
        REPO_URL = 'https://github.com/designking06/DevOpsProjCert.git'
        APP_DIR = '/home/ubuntu/DevOpsProjCert'  // repo clone location on slave
    }

    stages {
        stage('Prepare Slave Node') {
            steps {
                sshagent (credentials: [SSH_KEY]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} '
                            # Ensure directories exist
                            mkdir -p ${APP_DIR}
                            cd ${APP_DIR}

                            # Update system and install prerequisites
                            sudo apt update
                            sudo apt install -y git python3 python3-pip docker.io
                            
                            # Install ansible if not present
                            if ! command -v ansible-playbook > /dev/null; then
                                sudo apt install -y ansible
                            fi
                        '
                    """
                }
            }
        }

        stage('Clone Repository') {
            steps {
                sshagent (credentials: [SSH_KEY]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} '
                            if [ ! -d "${APP_DIR}/.git" ]; then
                                git clone ${REPO_URL} ${APP_DIR}
                            else
                                cd ${APP_DIR} && git pull
                            fi
                        '
                    """
                }
            }
        }

        stage('Install Docker via Ansible') {
            steps {
                sshagent (credentials: [SSH_KEY]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} '
                            cd ${APP_DIR}/ansible
                            ansible-playbook install_docker.yml -i ${EC2_HOST}, --user ${EC2_USER} --private-key /dev/null
                        '
                    """
                }
            }
        }

        stage('Deploy PHP Docker Container') {
            steps {
                sshagent(['WebAppPaC']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ubuntu@3.131.141.176 '
                        # Clone repo if it doesn'"'"'t exist, else pull latest
                        if [ ! -d /home/ubuntu/DevOpsProjCert ]; then
                            git clone https://github.com/designking06/DevOpsProjCert.git /home/ubuntu/DevOpsProjCert
                        else
                            cd /home/ubuntu/DevOpsProjCert && git pull
                        fi
        
                        cd /home/ubuntu/DevOpsProjCert
        
                        # Build Docker image from the docker/ subdirectory
                        sudo docker build -t webapp:latest -f docker/Dockerfile .
        
                        # Stop and remove old container if it exists
                        if [ \$(sudo docker ps -aq -f name=webapp) ]; then
                            sudo docker stop webapp
                            sudo docker rm webapp
                        fi
        
                        # Run new container
                        sudo docker run -d -p 80:80 --name webapp webapp:latest
                    '
                    """
                }
            }
        }
    }
    post {
            success {
                echo 'Pipeline completed successfully!'
            }
            failure {
                echo 'Pipeline failed. Check logs.'
            }
        }
    }