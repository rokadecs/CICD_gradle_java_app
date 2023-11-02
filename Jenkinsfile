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
        
       

        stage('Deploy to Kubernetes') {
        steps {
        script {
            def kubeconfigPath = '/root/.kube/config'
            def namespace = 'default'
            def deploymentName = 'deployment'
           

            dir('kubernetes/'){
            sh """
            curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.20.5/bin/linux/amd64/kubectl
            chmod u+x ./kubectl
            kubectl --kubeconfig=$kubeconfigPath --namespace=$namespace apply -f deployment.yaml
            """
            }
        }
        }
      }


        
    }
    
}