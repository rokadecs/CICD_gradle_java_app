pipeline{
    agent any
    environment{
        VERSION = "${env.BUILD_ID}"
    }
    stages{
        stage("Sonar Quality Check"){
            agent {
                docker { 
                image 'openjdk:11'
            }
            }
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
                    sh 'chmod +x gradlew'
                    sh './gradlew sonarqube'
                    }

                timeout(time: 1, unit: 'HOURS') {
                      def qg = waitForQualityGate()
                      if (qg.status != 'OK') {
                           error "Pipeline aborted due to quality gate failure: ${qg.status}"
                      }
                    }
                
                }
            }
            
        }

        stage("docker build & docker push"){
                steps{

                    script{
                        withCredentials([string(credentialsId: 'dockerNexus_pass', variable: 'docker_password')]) {

                        sh '''
                        docker build -t 34.93.183.110:8083/springapp:${VERSION} .
                        docker login -u admin -p $docker_password 34.93.183.110:8083
                        docker push 34.93.183.110:8083/springapp:${VERSION}
                        docker rmi 34.93.183.110:8083/springapp:${VERSION}
                        '''
                        }
                   }
                }

        }
    }
    
}