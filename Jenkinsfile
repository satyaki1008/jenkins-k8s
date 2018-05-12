//Jenkinsfile
node {
  withEnv(['PATH+JENKINSPATH=/home/jenkins/workspace']){
      
  

  stage('Preparation') {
    //sh 'sudo apt-get update && apt-get install -y apt-transport-https'
	//sh 'curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -'
	//sh 'sudo echo "deb http://apt.kubernetes.io/ kubernetes-xenial main">> /etc/apt/sources.list.d/kubernetes.list'
	//sh 'sudo apt-get update && apt-get install -y kubectl'
	//sh 'curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/darwin/amd64/kubectl'
	//sh 'chmod 777 ./kubectl'
	//sh 'mv kubectl ..'
	//sh 'ls -lsa /home/jenkins/workspace'
	sh 'uname -a'
	sh 'dpkg --print-architecture '
	tool name: 'Kubectl', type: 'com.cloudbees.jenkins.plugins.customtools.CustomTool'
	sh 'kubectl version'
  }

  stage('Integration') {
 	//sh 'curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/darwin/amd64/kubectl'
	//sh 'chmod +x ./kubectl'
	//sh 'mv kubectl ..'
	sh 'kubectl version'
    withKubeConfig([credentialsId: 'jenkins-deployer-credentials', serverUrl: 'https://104.155.31.202']) {
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
}