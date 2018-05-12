//Jenkinsfile
node {
  stage('Integration') {
    withKubeConfig([credentialsId: '<credential-id>', serverUrl: '<api-server-address>']) {
      sh 'cd /home/jenkins/workspace'
      sh 'kubectl create cm nodejs-app --from-file=src/ --namespace=myapp-integration --dry-run | kubectl apply -f -'
      sh 'kubectl apply -f deploy/ --namespace=myapp-integration'
      try{
		sh 'chmod +x tests/integration-tests.sh && ./tests/integration-tests.sh'
		println("Cleaning integration environment...")
		sh 'kubectl delete -f deploy --namespace=myapp-integration'
      	sh 'kubectl delete cm  nodejs-app --namespace=myapp-integration' 
        println("Integration stage finished.")                           
     
      }
      catch(Exception e) {
      	println("Integration stage aborted.")
	  	println("Cleaning integration environment...")
	  	sh 'kubectl delete -f deploy --namespace=myapp-integration'
      	sh 'kubectl delete cm  nodejs-app --namespace=myapp-integration'
      	println("Exiting...")
      	return                                       
      }

    }
  }
  stage('Production') {
    withKubeConfig([credentialsId: '<credential-id>', serverUrl: '<api-server-address>']) {
      sh 'cd /home/jenkins/workspace'
      sh 'kubectl create cm nodejs-app --from-file=src/ --namespace=myapp-production --dry-run | kubectl apply -f -' 
      sh 'kubectl apply -f deploy/ --namespace=myapp-production'
	  sh 'chmod +x tests/production-tests.sh && ./tests/production-tests.sh'                                       
    
    }
  }
}