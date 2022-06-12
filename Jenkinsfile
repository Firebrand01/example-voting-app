pipeline {

  agent none

  stages{
      stage("worker build"){
        when{
            changeset "**/worker/**"
          }

        agent{
          docker{
            image 'maven:3.6.1-jdk-8-slim'
            args '-v $HOME/.m2:/root/.m2'
          }
        }

        steps{
          echo 'Compiling worker app..'
          dir('worker'){
            sh 'mvn compile'
          }
        }
      }
      stage("worker test"){
        when{
          changeset "**/worker/**"
        }
        agent{
          docker{
            image 'maven:3.6.1-jdk-8-slim'
            args '-v $HOME/.m2:/root/.m2'
          }
        }
        steps{
          echo 'Running Unit Tets on worker app..'
          dir('worker'){
            sh 'mvn clean test'
           }

          }
      }
      stage("worker package"){
        when{
          branch 'master'
          changeset "**/worker/**"
        }
        agent{
          docker{
            image 'maven:3.6.1-jdk-8-slim'
            args '-v $HOME/.m2:/root/.m2'
          }
        }
        steps{
          echo 'Packaging worker app'
          dir('worker'){
            sh 'mvn package -DskipTests'
            archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
          }

        }
      }


        stage('worker-dockerpackage') {
      
            steps {
                echo 'Packaging worket app with docker'

                script{
                         docker.withRegistry('https://index.docker.io/v1', 'dockerhub') {
                         def workerImage = docker.build("firebrand01/worker:v${env.BUILD_ID}", "./worker")
                         workerImage.push()
                         workerImage.push("${env.BRANCH_NAME}")

      
                             }
                }
                
            }
        }


        stage('result build') {


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
        stage('result test') {

            when {
                
                changeset "**/result/**"
            }
            steps {
                echo 'Running Unit Test on result app'
                   dir('result') {
                    sh 'npm install'
                    sh 'npm test'

                }
                
            }
        }

        stage('result-docker-package') {

            agent any
      
            steps {
                echo 'Packaging worket app with docker'

                script{
                         docker.withRegistry('https://index.docker.io/v1', 'dockerhub') {
                         def workerImage = docker.build("firebrand01/worker:v${env.BUILD_ID}", "./result")
                         workerImage.push()
                         workerImage.push("${env.BRANCH_NAME}")

      
                             }
                }
                
            }
         

     
        stage('vote build') {


              agent {
                            docker {
                                image 'python:2.7.16-slim'
                                args '--user root'
                            }
                        }


            when{
                changeset "**/vote/**"
            }


            steps {
                echo 'Compiling vote app'
                dir('vote') {
                    sh 'pip install -r requirements.txt'

                }
                
            }
        } 
/*         stage('test') {

            when {
                
                changeset "vote"
            }

                  agent {
                            docker {
                                image 'python:2.7.16-slim'
                                args '--user root'
                            }
                        }


            steps {
                echo 'Running Unit Test on vote app'
                   dir('vote') {
                    sh 'nosetests -v'
               

                }
                
            }
        } */

             stage('vote docker-package') {

            agent any

        when{
            changeset "**/vote/**"
           
          }
      
            steps {
                echo 'Packaging vote app with docker'

                script{
                         docker.withRegistry('https://index.docker.io/v1', 'dockerhub') {
                         def workerImage = docker.build("firebrand01/vote:v${env.BUILD_ID}", "./vote")
                         workerImage.push()
                         workerImage.push("${env.BRANCH_NAME}")
                         workerImage.push("latest")
                             }
                }
                
            }
        }  
        
    

        
    }


    }
    post {
        always {
            echo 'Build pipeline for worker run is complete..'
        }
        failure {
            slackSend (channel: "instavote-cd", message: "Build Failed - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL} | Open>)")
        }

        success {
            slackSend (channel: "instavote-cd", message: "Build Succeeded - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL} | Open>)")
        }



        }   
    
    
}
