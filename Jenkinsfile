pipeline{
    agent any

        environment {
            DOCKERHUB_USER="emrverskn"
            APP_REPO_NAME="todo-app"
            DB_VOLUME="myvolume"
            NETWORK="mynetwork"
            POSTGRES_PASSWORD="Pp123456789"
            
        }
    stages {
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
                withCredentials([string(credentialsId: 'DockerHub-Token', variable: 'DOCKERHUB_TOKEN')]) {
                sh 'docker login -u emrverskn -p $DOCKERHUB_TOKEN'
                sh 'docker push "$DOCKERHUB_USER/$APP_REPO_NAME:postgre"'
                sh 'docker push "$DOCKERHUB_USER/$APP_REPO_NAME:nodejs"'
                sh 'docker push "$DOCKERHUB_USER/$APP_REPO_NAME:react"'
            }
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
                echo 'Deploying the Postgresql'
                withCredentials([string(credentialsId: 'Postgres-Password', variable: 'POSTGRES_PASSWORD')]) {
                sh 'docker run --name postgres -p 5432:5432 -v $DB_VOLUME:/var/lib/postgresql/data --network $NETWORK -e POSTGRES_PASSWORD=$POSTGRES_PASSWORD --restart always -d $DOCKERHUB_USER/$APP_REPO_NAME:postgre'
             }
        }
    }
        stage('wait the postgres container') {
            steps {
                script {
                    echo 'Waiting for the containers'
                    sh 'sleep 60s'
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

        stage('Approve all stages'){
            steps{
                timeout(time:5, unit:'DAYS'){
                    input message:'Approve terminate'
                }
            }
        }
    }
    
    post {
        always {
            echo 'Cleaning up'
            script {
                sh 'docker rm -f $(docker ps -aq)'
                sh 'docker network rm $NETWORK'
                sh 'docker volume rm $DB_VOLUME'
                sh 'docker rmi -f $DOCKERHUB_USER/$APP_REPO_NAME:postgre $DOCKERHUB_USER/$APP_REPO_NAME:nodejs $DOCKERHUB_USER/$APP_REPO_NAME:react'
            }
        }

        success {
            echo 'Pipeline executed successfully'
            sh 'echo "SUCCESS"'
        }

        failure {
            echo 'Pipeline failed. Cleaning up containers, images, network, and volume.'
            sh 'echo "FAILURE"'
        }
    }
}