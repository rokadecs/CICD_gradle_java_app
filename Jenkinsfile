pipeline{
    agent any
    environment{
        VERSION = "${env.BUILD_ID}"
    }
    stages{
        stage("Sonar Quality Check"){
            agent { 
                docker { image 'openjdk:11' }
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

        stage("docker build & docker push to Nexus Repo"){
                steps{

                    script{
                        withCredentials([string(credentialsId: 'dockerNexus_pass', variable: 'docker_password')]) {

                        sh '''
                        docker build -t 34.93.74.138:8083/springapp:${VERSION} .
                        docker login -u admin -p $docker_password 34.93.74.138:8083
                        docker push 34.93.74.138:8083/springapp:${VERSION}
                        docker rmi 34.93.74.138:8083/springapp:${VERSION}
                        '''
                        }
                   }
                }

        }

        //stage("pushing helmcharts to Nexus repo"){
        //        steps{

        //            script{
        //               withCredentials([string(credentialsId: 'dockerNexus_pass', variable: 'nexus_password')]) {
        //              dir('kubernetes/'){

        //                  sh '''
        //                 helmversion=$( helm show chart myapp | grep version | cut -d':' -f 2 | tr -d ' ' )
        //                tar -czvf myapp-${helmversion}.tgz myapp/
        //                 curl -u admin:$nexus_password http://34.93.74.138:8081/repository/helm-hosted/ --upload-file myapp-${helmversion}.tgz -v
        //               '''
        //              }      
                        
        //                }
        //           }
        //        }

        //}
        
        stage('Pull Artifact from Nexus') {
        steps {
        script {
            def nexusUrl = 'http://34.93.74.138:8081/service/rest/repository/browse/helm-hosted/myapp/0.2.0/'
            def artifactName = 'myapp-0.2.0.tgz'
            def nexusCredentialsId = 'dockerNexus_pass'

            withCredentials([usernamePassword(credentialsId: nexusCredentialsId, passwordVariable: 'NEXUS_PASSWORD')]) {
                sh """
                curl -u admin:$NEXUS_PASSWORD -o $artifactName $nexusUrl/$artifactName
                """
            }
          }
         }
    
        }

        stage('Deploy to Kubernetes') {
        steps {
        script {
            def kubeconfigPath = '/path/to/your/kubeconfig'
            def namespace = 'your-namespace'
            def deploymentName = 'your-deployment'
            def artifactName = 'your-artifact.jar'

            sh """
            kubectl --kubeconfig=$kubeconfigPath --namespace=$namespace apply -f your-deployment-manifest.yaml
            """
            }
        }
      }


        
    }
    
}