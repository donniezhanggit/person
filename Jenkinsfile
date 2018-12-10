#!/usr/bin/env groovy

pipeline {
    agent any

    triggers {
        pollSCM('*/15 * * * *')
    }

    options { disableConcurrentBuilds() }

    stages {
        stage('Permissions') {
            steps {
                sh 'chmod 775 *'
            }
        }
		
		stage('Cleanup') {
            steps {
                sh './gradlew --no-daemon clean'
            }
        }

        stage('Code analysis') {
            steps {
                sh './gradlew --no-daemon checkstyleMain checkstyleTest findbugsMain findbugsTest pmdMain pmdTest cpdCheck'
            }
        }

        stage('Test') {
            steps {
                sh './gradlew --no-daemon check'
            }
            post {
                always {
                    junit 'build/test-results/test/*.xml'
                }
            }
        }

        stage('Build') {
            steps {
                sh './gradlew --no-daemon build'
            }
        }

        stage('Update Docker UAT image') {
            when { branch "master" }
            steps {
                sh '''
					docker login -u "amritendudockerhub" -p "Passw1rd"
                    docker build --no-cache -t person .
                    docker tag person:latest amritendudockerhub/person:latest
                    docker push amritendudockerhub/person:latest
					docker rmi $(docker images -f “dangling=true” -q)
                '''
            }
        }

        stage('Update UAT container') {
            when { branch "master" }
            steps {
                sh '''
					docker login -u "amritendudockerhub" -p "Passw1rd"
                    docker pull amritendudockerhub/person:latest
					docker stop person
					docker rm person
					docker run -p 9090:9090 --name person -t -d amritendudockerhub/person
                '''
            }
        }

        stage('Release Docker image') {
            when { buildingTag() }
            steps {
                sh '''
					docker login -u "amritendudockerhub" -p "Passw1rd"
                    docker build --no-cache -t person .
                    docker tag person:latest amritendudockerhub/person:${TAG_NAME}
                    docker push amritendudockerhub/person:${TAG_NAME}
					docker rmi $(docker images -f “dangling=true” -q)
               '''
            }
        }
    }
}
