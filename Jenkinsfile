pipeline{
    agent any

    }

    environment {
        DOCKERHUB_USER="emrverskn"
        APP_REPO_NAME="todo-app"
        DB_VOLUME="myvolume"
        NETWORK="mynetwork"
        POSTGRES_PASSWORD="Pp123456789"
        
    }
        stage('Build App Docker Image') {
            steps {
                sh 'docker build --force-rm -t "$DOCKERHUB_USER/$APP_REPO_NAME:postgre" -f ./database/Dockerfile .'
                sh 'docker build --force-rm -t "$DOCKERHUB_USER/$APP_REPO_NAME:nodejs" -f ./server/Dockerfile .'
                sh 'docker build --force-rm -t "$DOCKERHUB_USER/$APP_REPO_NAME:react" -f ./client/Dockerfile .'
                sh 'docker image ls'
            }
        }

        stage('Push Image to DockerHub Repo') {
            steps {
                echo 'Pushing App Image to DockerHub Repo'
                sh 'docker login -u emrverskn -p dckr_pat_gjCCuQYq8ADyjCSYzbdfsg4YqwM'
                sh 'docker push "$DOCKERHUB_USER/$APP_REPO_NAME:postgre"'
                sh 'docker push "$DOCKERHUB_USER/$APP_REPO_NAME:nodejs"'
                sh 'docker push "$DOCKERHUB_USER/$APP_REPO_NAME:react"'
            }
        }

        stage('Create Volume') {
            steps {
                echo 'Creating Volume'
                sh 'docker volume create $DB_VOLUME'
             }
        }

        stage('Create Network') {
            steps {
                echo 'Creating Network'
                sh 'docker network create $NETWORK'
             }
        }

        stage('Deploy the Database') {
            steps {
                echo 'Deploy the Postgresql'
                sh 'docker run --name postgres -p 5432:5432 -v $DB_VOLUME:/var/lib/postgresql/data --network $NETWORK -e POSTGRES_PASSWORD=$POSTGRES_PASSWORD --restart always -d $DOCKERHUB_USER/$APP_REPO_NAME:postgre'
             }
        }

        stage('wait the postgres container') {
            steps {
                script {
                    echo 'Waiting for the containers'
                    sh 'sleep 120s'
                }
            }
        }

        stage('Deploy the Server') {
            steps {
                echo 'Deploying the Nodejs'
                sh 'docker run --name nodejs -p 5000:5000 --network $NETWORK --restart always -d $DOCKERHUB_USER/$APP_REPO_NAME:nodejs'
             }
        }

        stage('wait the nodejs container') {
            steps {
                script {
                    echo 'Waiting for the containers'
                    sh 'sleep 60s'
                }
            }
        }

        stage('Deploy the Client') {
            steps {
                echo 'Deploying the React'
                sh 'docker run --name react -p 3000:3000 --network $NETWORK --restart always -d $DOCKERHUB_USER/$APP_REPO_NAME:react'
             }
        }

        stage('wait the nodejs container') {
            steps {
                script {
                    echo 'Waiting for the containers'
                    sh 'sleep 60s'
                }
            }
        }


        stage('Delete containers and images'){
            steps{
                timeout(time:5, unit:'DAYS'){
                    input message:'Approve terminate'
                }
                sh 'docker rm -f $(docker ps -aq)'
                sh 'docker network rm $NETWORK'
                sh 'docker volume rm $DB_VOLUME'
                sh 'docker rmi -f $DOCKERHUB_USER/$APP_REPO_NAME:postgre $DOCKERHUB_USER/$APP_REPO_NAME:nodejs $DOCKERHUB_USER/$APP_REPO_NAME:react'
            }
        }


