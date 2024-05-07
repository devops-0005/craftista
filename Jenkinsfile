pipeline {
  agent none
  stages {
    stage('Voting Build') {
      agent {
        docker {
          image 'maven:3.9.6-eclipse-temurin-17-alpine'
        }

      }
      steps {
        echo 'compiling the voting app...'
        dir(path: 'voting') {
          sh 'mvn compile'
        }

      }
    }

    stage('Voting Test') {
      agent {
        docker {
          image 'maven:3.9.6-eclipse-temurin-17-alpine'
        }

      }
      steps {
        dir(path: 'voting') {
          sh 'mvn clean test '
        }

      }
    }

    stage('Voting Package') {
      parallel {
        stage('Voting Package') {
          agent {
            docker {
              image 'maven:3.9.6-eclipse-temurin-17-alpine'
            }

          }
          when {
            branch 'main'
          }
          steps {
            dir(path: 'voting') {
              sh 'mvn package -DskipTests'
            }

            archiveArtifacts '**/target/*.jar'
          }
        }

        stage('Voting Image B&P') {
          agent any
          when {
            branch 'main'
          }
          steps {
            script {
              docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                def commitHash = env.GIT_COMMIT.take(7)
                def dockerImage = docker.build("initcron/craftista-voting:${commitHash}", "./voting")
                dockerImage.push()
                dockerImage.push("latest")
                dockerImage.push("dev")
              }
            }

          }
        }

      }
    }

    stage('Frontend Build') {
      agent {
        docker {
          image 'node:latest'
        }

      }
      steps {
        dir(path: 'frontend') {
          sh 'npm install'
        }

      }
    }

    stage('Frontend Test') {
      agent {
        docker {
          image 'node:latest'
        }

      }
      steps {
        dir(path: 'frontend') {
          sh '''npm install
npm test'''
        }

      }
    }

    stage('Frontend Image B&P') {
      agent any
      steps {
        script {
          docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
            def commitHash = env.GIT_COMMIT.take(7)
            def dockerImage = docker.build("initcron/craftista-frontend:${commitHash}", "./voting")
            dockerImage.push()
            dockerImage.push("latest")
            dockerImage.push("dev")
          }
        }

      }
    }

  }
  tools {
    maven 'Maven 3.9.6'
  }
  post {
    always {
      echo 'completed the pipeline for craftista....'
    }

  }
}