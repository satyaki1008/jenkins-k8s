//Jenkinsfile
node {
  //Adding kubectl path to PATH environment variable
  //withEnv(["PATH+JENKINSPATH=/kubectl/bin'}"]){

  stage('Preparation') {
    //Installing kubectl in Jenkins agent
    sh 'curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl'
	sh 'chmod +x ./kubectl && mv kubectl /usr/local/sbin'
	sh 'ls -lsa /usr/local/bin'
	sh 'ls -lsa /usr/local/sbin'
	sh 'ls -lsa'
	//tool name: 'Kubectl', type: 'com.cloudbees.jenkins.plugins.customtools.CustomTool'
	git url:'https://bitbucket.org/advatys/jenkins-pipeline.git'
  }

  stage('Integration') {
	//tool name: 'Kubectl', type: 'com.cloudbees.jenkins.plugins.customtools.CustomTool'
	//sh 'ls -lsa /home/jenkins/tools/com.cloudbees.jenkins.plugins.customtools.CustomTool/Kubectl/kubectl/bin'


	
	sh 'kubectl version'
    withKubeConfig([credentialsId: 'jenkins-deployer-credentials', serverUrl: 'https://104.155.31.202']) {
      
      sh 'kubectl create cm nodejs-app --from-file=	jenkins-pipeline/src/ --namespace=myapp-integration --dry-run | kubectl apply -f -'
      sh 'kubectl apply -f deploy/ --namespace=myapp-integration'
      try{
      	//Gathering Node.js app's external IP address
        sh 'export MYAPP_IP=$(kubectl get svc --namespace=myapp-integration -o jsonpath="{.items[?(@.metadata.name==\'nginx-reverseproxy-service\')].status.loadBalancer.ingress[*].ip}")'
        
        //Executing tests 
		sh 'chmod +x tests/integration-tests.sh && ./tests/integration-tests.sh $MYAPP_IP'
		
		//Cleaning the integration environment
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
    withKubeConfig([credentialsId: 'jenkins-deployer-credentials', serverUrl: 'https://104.155.31.202']) {
      
      sh 'kubectl create cm nodejs-app --from-file=src/ --namespace=myapp-production --dry-run | kubectl apply -f -' 
      sh 'kubectl apply -f deploy/ --namespace=myapp-production'
      
      //Gathering Node.js app's external IP address
      sh 'export MYAPP_IP=$(kubectl get svc --namespace=myapp-production -o jsonpath="{.items[?(@.metadata.name==\'nginx-reverseproxy-service\')].status.loadBalancer.ingress[*].ip}")'
      
      //Executing tests 
	  sh 'chmod +x tests/production-tests.sh && ./tests/production-tests.sh $MYAPP_IP'                                         
    }
  }
  //}
}