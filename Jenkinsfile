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

               ''' timeout(time: 1, unit: 'HOURS') {
                      def qg = waitForQualityGate()
                      if (qg.status != 'OK') {
                           error "Pipeline aborted due to quality gate failure: ${qg.status}"
                      }
                    }
                '''
                
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

        stage("pushing helmcharts to Nexus repo"){
                steps{

                    script{
                        withCredentials([string(credentialsId: 'dockerNexus_pass', variable: 'nexus_password')]) {
                      dir('kubernetes/'){

                          sh '''
                         helmversion=$( helm show chart myapp | grep version | cut -d':' -f 2 | tr -d ' ' )
                         tar -czvf myapp-${helmversion}.tgz myapp/
                         curl -u admin:$nexus_password http://34.93.183.110:8081/repository/helm-hosted/ --upload-file myapp-${helmversion}.tgz -v
                        '''
                      }      
                        
                        }
                   }
                }

        }

        stage('Deploying application on k8s cluster') {
            steps {
               script{
                   withCredentials([kubeconfigFile(credentialsId: 'kubernetes-config', variable: 'KUBECONFIG')]) {
                        dir('kubernetes/') {
                          sh 'helm upgrade --install --set image.repository="34.125.214.226:8083/springapp" --set image.tag="${VERSION}" myjavaapp myapp/ ' 
                        }
                    }
               }
            }
        }


        
    }
    
}