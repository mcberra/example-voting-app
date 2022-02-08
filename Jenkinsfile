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
        dir(path: 'worker') {
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
        dir(path: 'worker') {
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
      steps {
        echo 'Starting PACKGING......'
        dir(path: 'worker') {
          sh 'mvn package -DskipTests'
        }

        archiveArtifacts(artifacts: '**/target/*.jar', fingerprint: true)
        echo '### FINISHED PACKAGING ###'
      }
    }

    stage('Worker-Docker-package') {
      agent any
      steps {
        echo 'Starting PACKAGING the app with docker.....'
        script {
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
        dir(path: 'result') {
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
        dir(path: 'result') {
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
        script {
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
        dir(path: 'vote') {
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
        dir(path: 'vote') {
          sh 'pip install -r requirements.txt'
          sh 'nosetests -v'
        }

      }
    }

    stage('Vote-Docker-package') {
      agent any
      steps {
        echo 'Starting PACKAGING the app with docker.....'
        script {
          docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
            def workerImage = docker.build("mcberra/vote:v${env.BUILD_ID}", "./vote")
            workerImage.push()
            workerImage.push("${env.BRANCH_NAME}")
          }
        }

      }
    }


    stage('Sonarqube') {
      agent any
 #     when{
 #       branch 'master'
 #     }
      tools {
        jdk "JDK11" // the name you have given the JDK installation in Global Tool Configuration
      }

      environment{
        sonarpath = tool 'SonarScanner'
      }

      steps {
            echo 'Running Sonarqube Analysis......................................................................................................................'
            withSonarQubeEnv('sonar-instavote') {
              sh "${sonarpath}/bin/sonar-scanner -Dproject.settings=sonar-project.properties -Dorg.jenkinsci.plugins.durabletask.BourneShellScript.HEARTBEAT_CHECK_INTERVAL=86400"
            }
      }
    }


    stage("Quality Gate") {
        steps {
            timeout(time: 1, unit: 'HOURS') {
                // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                // true = set pipeline to UNSTABLE, false = don't
                waitForQualityGate abortPipeline: true
            }
        }
    }



    stage('Deploy to Dev') {
      agent any
      when {
        branch 'master'
      }
      steps {
        sh 'docker-compose up -d'
      }
    }

  }
  post {
    always {
      echo '##################### This pipeline run is completed!'
    }

  }
}
