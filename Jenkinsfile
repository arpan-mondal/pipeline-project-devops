pipeline{
    agent any
    environment {
        VERSION = "${env.BUILD_ID}"
    }

    stages{

        stage('sonar quality status')
        agent{
            docker {
                image 'maven'
            }
        }
        steps{
            scripts{
               withSonarQubeEnv(credentialsId: 'Server-Authentication-Token')

               sh 'mvn clean package sonar:sonar'
            }
        }
     
       stage('Quality Gate Status'){
        
        steps{
            scripts{
                            // You dont have to remember this, just generate from pipeline scripts
                waitForQualityGate abortPipeline: false, credentialsId: 'Server-Authentication-Token' 

            }
          }
       }
       stage('docker build & docker push to Nexus repo'){

        steps{
            
            script{
                          //multi-line shell script, as we plan to use nexus repo we need to login it and create repository.
                //to use nexus repo password
                withCredentials([string(credentialsID: 'ID', variable: 'variable_name')]){
                sh '''  
                docker build -t <nexus ip and port>/springapp:${VERSION} .

                docker login -u admin -p $variable_name <nexus ip and port>

                docker push <nexus ip and port>/springapp:${VERSION}
                
                docker rmi <nexus ip and port>/springapp:${VERSION}
                '''
              }
            }
          }
       }
       stage('Identifying misconfigs using datree in Helm Charts'){

        steps{
            scripts{
                dir('kubernetes/myapp/')
                { //set environmnet variable : copy token from token management : choose withEnv and set the variable
                   
                   withEnv(['DATREE_TOKEN=tokan-value']) {
                    sh 'helm datree test .'
                }
              }
            }
          }
       }
       stage('Pushing the helm charts to nexus repo'){
        steps{
            script{
                withCredentials([string(credentialsID: 'ID', variable: 'variable_name')]){
                 dir('kubernetes/'){
                    sh '''
                    helmversion=$(helm show charts myapp | grepversion | cut -d: -f 2 | tr -d '  ')
                    tar -czvf myapp-${helmversion}.tgz myapp/
                      curl -u admin:$variable_name http://nexus_machine_ip:8081/repository/helm-hosted/ --upload-file myapp-${helmversion}.tgz -v 
                 '''
              }
             }
           }
         }
       }
       stage('Deploy to Kubernetes'){
        steps{
            script{
                withCredentials([string(credentialsID: 'KUBECONFIG', variable: 'KUBECONFIG')]){
                  sh '''
                    export KUBECONFIG=$KUBECONFIG
                    kubectl config use-context <context-name>
                    kubectl set image deployment/myapp myapp=<nexus ip and port>/springapp:${VERSION}
                    helm upgrade myapp kubernetes/myapp/ --version ${helmversion} --set image.repository=<nexus ip and port>/springapp --set image.tag=${VERSION}
                  '''
              }
            }
          }
       }
    }
  
        post {
		always {
			mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "sender's-email-address@gmail.com";  
		}
	}
    
}

