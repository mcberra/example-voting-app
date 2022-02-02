pipeline {
    agent none

    stages {

        stage('Worker-Build') { 
    	  agent {
            docker {
              image 'maven:3.6.1-jdk-8-alpine'
              args '-v $HOME/.m2:/root/.m2'
            }
        }
            steps {
                echo 'Starting BUILD.....'
                echo 'Compiling worker app'
		dir('worker'){
		    sh 'mvn compile' 
		}
                echo '### FINISHED BUILDING ###'
            }
        }

        stage('Worker-Test') { 
    	    agent {
              docker {
                image 'maven:3.6.1-jdk-8-alpine'
                args '-v $HOME/.m2:/root/.m2'
              }
          }
            steps {
                echo 'Starting TEST.....'
		dir('worker'){
		    sh 'mvn clean test' 
		}
                echo '### FINISHED TESTING ###'
            }
        }

        stage('Worker-Package') { 
    	    agent {
              docker {
                image 'maven:3.6.1-jdk-8-alpine'
                args '-v $HOME/.m2:/root/.m2'
              }
          }
	    when {
		branch 'master'
		changeset "**/worker/**"
	    }
            steps {
                echo 'Starting PACKGING......'
		dir('worker'){
		    sh 'mvn package -DskipTests'
		}
		archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
                echo '### FINISHED PACKAGING ###' 
            }
        }

        stage('Worker-Docker-package') {
            agent any 
	    when {
		changeset "**/worker/**"
                branch 'master'
	    }
            steps {
                echo 'Starting PACKAGING the app with docker.....'
                script{
                  docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                     def workerImage = docker.build("mcberra/worker:v${env.BUILD_ID}", "./worker")
		     workerImage.push()
                     workerImage.push("${env.BRANCH_NAME}")
                  }
                }
            }
        }

        stage('Resul-Build') {
          agent {
            docker {
              image 'node:14-alpine'
              }
            }
            steps {
                echo 'Compiling Result app'
                dir('result'){
                    sh 'npm install'
                }
                echo '### FINISHED BUILDING ###'
            }
        }
        stage('Result-Test') {
          agent {
            docker {
              image 'node:14-alpine'
              }
            }
            steps {
                dir('result'){
                    sh 'npm install'
                    sh 'npm test'
                }
                echo '### FINISHED TESTING ###'
            }
        }

        stage('Result-Docker-package') {
            agent any
            steps {
                echo 'Starting PACKAGING the app with docker.....'
                script{
                  docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                     def workerImage = docker.build("mcberra/result:v${env.BUILD_ID}", "./result")
                     workerImage.push()
                     workerImage.push("${env.BRANCH_NAME}")
                  }
                }
            }
        }
        stage('Vote-Build') {

          agent {
            docker {
              image 'python:2.7.16-slim'
              args '--user root'
                }
              }

            steps {
                echo 'Compiling vote app'
                dir('vote'){
                    sh 'pip install -r requirements.txt'
                }
                echo '### FINISHED BUILDING ###'
            }
        }

        stage('Vote-Test') {
            agent {
              docker {
                image 'python:2.7.16-slim'
                args '--user root'
              }
            }
            steps {

                echo '###  TESTING VOTE APP ###'
                dir('vote'){
                    sh 'pip install -r requirements.txt'
                    sh 'nosetests -v'
                }
            }
        }

        stage('Vote-Docker-package') {
            agent any
            steps {
                echo 'Starting PACKAGING the app with docker.....'
                script{
                  docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                     def workerImage = docker.build("mcberra/vote:v${env.BUILD_ID}", "./vote")
                     workerImage.push()
                     workerImage.push("${env.BRANCH_NAME}")
                  }
                }
            }
        }



    }
        post { 
            always { 
                echo '##################### This pipeline run is completed!'
        }
    }
}