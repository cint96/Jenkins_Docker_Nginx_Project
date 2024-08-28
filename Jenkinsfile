pipeline {
    agent any

    parameters {
        string(name: 'container_name', defaultValue: 'web_server', description: 'Name of the Docker container')
        string(name: 'image_name', defaultValue: 'webserver', description: 'Name of the Docker image')
        string(name: 'version', defaultValue: 'v1', description: 'Version or tag of the Docker image')
    }

    environment {
        DOCKERHUB_CREDENTIALS = credentials('cinta96_dockerhub_token') // Use the ID from Jenkins credentials
    }

    stages {
        stage('Clean-up') {
            steps {
                script {
                    def containerExists = sh(script: "docker ps -a -f name=${container_name} --format '{{.Names}}'", returnStdout: true).trim()
                    if (containerExists == container_name) {
                        echo 'Stopping and removing the container...'
                        sh '''
                            docker stop ${container_name}
                            docker rm ${container_name}
                        '''
                    } 
                    else {
                        echo 'Container ${container_name} does not exist or is not running.'
                    }
                }
                // Always attempt to remove the image if it exists
                sh '''
                    docker rmi ${image_name}:${version} || echo "Image ${image_name}:${version} does not exist."
                '''
            }
        }

        stage('Pull-changes') {
            steps {
                sh 'git pull origin main'
            }
        }

        stage('Build-Image') {
            steps {
                sh'docker build -t ${image_name}:${version} .'
                echo'build ${image_name} image'
            }
        }

        stage('Tag-Image') {
            steps {
                sh'docker tag ${image_name}:${version} cinta96/${image_name}:${version}'
            }
        }

        stage('Docker-Login') {
            steps {
                sh "echo ${DOCKERHUB_CREDENTIALS_PSW} | docker login -u ${DOCKERHUB_CREDENTIALS_USR} --password-stdin"
            }
        }

        stage('Push-Remove Image') {
            steps {
                sh'''
                echo " Push ${image_name}:${version} image to docker hub "
                docker push cinta96/${image_name}:${version} 
                echo " remove ${image_name}:${version} from local "
                docker rmi ${image_name}:${version}
                '''
            }
        }

        stage('Build-Container') {
            steps {
                sh'docker run -t --name ${container_name} -p 81:80 cinta96/${image_name}:${version}'
            }
        }

        stage('Display-URL') {
            steps {
                sh'''
                def IpAddress = docker inspect ${container_name} | awk '/IPAddress/ {gsub(/[",]/, "", $2); print $2}' | tail -n1
                echo "URL - http://${IpAddress}"
                '''
            }
        }
    }
}