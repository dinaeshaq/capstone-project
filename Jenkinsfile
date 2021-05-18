node {
    def registry1 = 'dinaeshaq/capstone-blue'
    def registry2 = 'dinaeshaq/capstone-green'
    stage('Checking out git repo') {
      echo 'Checkout...'
      checkout scm
    }
    stage('Checking environment') {
      echo 'Checking environment...'
      sh 'git --version'
      echo "Branch: ${env.BRANCH_NAME}"
      sh 'docker -v'
    }
    stage("Linting") {
      echo 'Linting...'
      sh '/usr/bin/hlint blue/Dockerfile'
      sh '/usr/bin/hlint green/Dockerfile'
    }
    stage('Building image blue') {
	    echo 'Building Docker image blue...'
      withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
	     	sh "sudo docker login -u ${env.dockerHubUser} -p ${env.dockerHubPassword}"
	     	sh "sudo docker build -t ${registry1} blue/."
	     	sh "sudo docker tag ${registry1} ${registry1}"
	     	sh "sudo docker push ${registry1}"
      }
    }
    stage('Building image green') {
	    echo 'Building Docker image green...'
      withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
	     	sh "sudo docker login -u ${env.dockerHubUser} -p ${env.dockerHubPassword}"
	     	sh "sudo docker build -t ${registry2} green/."
	     	sh "sudo docker tag ${registry2} ${registry2}"
	     	sh "sudo docker push ${registry2}"
      }
    }
    
    stage('Deploying to AWS EKS') {
      echo 'Deploying to AWS EKS...'
      dir ('./') {
        withAWS(credentials: '115633711417', region: 'us-west-2') {
            sh "pwd" 
            sh "whoami"
            sh "aws eks --region us-west-2 update-kubeconfig --name bn-prod"
            echo 'get kluster services'
            sh "/var/lib/jenkins/bin/kubectl get services"   
            sh "/var/lib/jenkins/bin/kubectl apply -f blue/blue-controller.json"
            sh "/var/lib/jenkins/bin/kubectl apply -f green/green-controller.json"
            sh "/var/lib/jenkins/bin/kubectl apply -f ./blue-green-service.json"
            sh "/var/lib/jenkins/bin/kubectl get nodes"
            sh "/var/lib/jenkins/bin/kubectl get pods"
        }
      }
    }
    
    stage("Cleaning up") {
      echo 'Cleaning up...'
      sh "sudo docker system prune -a -f "
    }
}
