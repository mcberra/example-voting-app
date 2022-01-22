pipeline {
    agent none

    stages {

        stage('Build') {
 
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

        stage('Test') {
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

        stage('Docker-package') {
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
            echo 'This pipeline run is completed!'
        }
    }
}
