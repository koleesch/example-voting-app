pipeline {
    agent none
    stages {
        stage('worker-build') {
            agent {
                docker {
                    image 'maven:3.8.6-eclipse-temurin-18-alpine'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            when{
                changeset "**/worker/**"
            }
            steps {
                echo 'Compiling worker app'
                dir('worker') {
                    sh 'mvn compile'
                }
            }
        }
        stage('worker-test') {
            agent {
                docker {
                    image 'maven:3.8.6-eclipse-temurin-18-alpine'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            when{
                changeset "**/worker/**"
            }
            steps {
                echo 'Running Unit Tests on worker app'
                dir('worker') {
                    sh 'mvn clean test'
                }
            }
        }
        stage('worker-package') {
            agent {
                docker {
                    image 'maven:3.8.6-eclipse-temurin-18-alpine'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            when{
                branch 'master'
                changeset "**/worker/**"
            }
            steps {
                echo 'Packaging worker app'
                dir('worker') {
                    sh 'mvn package -DskipTests'
                    archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
                }
            }
        }
        stage('worker-docker-package') {
            agent any
            when{
                branch 'master'
                changeset "**/worker/**"
            }
            steps {
                echo 'Packaging worker app with docker'
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                        def workerImage = docker.build("koleesch/worker:v${env.BUILD_ID}", "./worker")

                        workerImage.push()
                        workerImage.push("${env.BRANCH_NAME}")
                    }
                }
            }
        stage('result-build') {
            agent {
                docker{
                    image 'node:lts-alpine'
                }        
            }
            when{
                changeset "**/result/**"
            }
            steps {
                echo 'Compiling result app'
                dir('result') {
                    sh 'npm install'
                }
            }
        }
        stage('result-test') {
            agent {
                docker{
                    image 'node:lts-alpine'
                }        
            }
            when{
                changeset "**/result/**"
            }
            steps {
                echo 'Running Unit Tests on result app'
                dir('result') {
                    sh 'npm install'
                    sh 'npm test'
                }
            }
        }       
        stage('result-docker-package') {
            agent any
            when{
                branch 'master'
                changeset "**/result/**"
            }
            steps {
                echo 'Packaging result app with docker'
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                        def workerImage = docker.build("koleesch/result:v${env.BUILD_ID}", "./result")
                        workerImage.push()
                        workerImage.push("${env.BRANCH_NAME}")
                    }
                }
            }
        }  
        stage('vote-build') {
            agent {
                docker {
                    image 'python:2.7.18-slim'
                    args '--user root'
                }
            }

            steps {
                echo 'Build vote app'
                dir ('vote') {
                    sh 'pip install -r requirements.txt'
                }
            }
        }
        stage('vote-test') {
            agent {
                docker {
                    image 'python:2.7.18-slim'
                    args '--user root'
                }
            }

            steps{
                echo 'Testing vote app'
                dir ('vote') {
                    sh 'pip install -r requirements.txt'
                    sh 'nosetests -v'
                }
            }
        }
        stage('vote-docker-package') {
            agent any
            when{
                branch 'master'
                changeset "**/vote/**"
            }
            steps {
                echo 'Packaging vote app with docker'
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                        def workerImage = docker.build("koleesch/vote:v${env.BUILD_ID}", "./vote")
                        workerImage.push()
                        workerImage.push("${env.BRANCH_NAME}")
                    }
                }
            }
        }        
    }
    post {
        always {            
            echo 'mono pipeline for worker, vote and result app is completed...'
        }
    }
}