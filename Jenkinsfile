ansible_server_private_ip="192.168.0.14"
kubernetes_server_private_ip="192.168.0.12"
REPO_URL="https://github.com/akramLh005/tp3-devops-k8s.git"
BRANCH= "main"   
node{
    stage('Git checkout'){
        git branch: "${BRANCH}", url: "${REPO_URL}"
    }
    
     //all below sshagent variables created using Pipeline syntax
stage('Sending Dockerfile to Ansible server') {
    sshagent(['ansible-server']) {
        // Disable pseudo-terminal allocation with -T and test connection
        sh "ssh -o StrictHostKeyChecking=no vagrant@${ansible_server_private_ip} 'echo Connection established'"

        // Use scp with the same credentials
        sh "scp -o StrictHostKeyChecking=no /var/lib/jenkins/workspace/tp3-devops-k8s/* vagrant@${ansible_server_private_ip}:/home/vagrant"
    }
}

    
    stage('Docker build image'){
        sshagent(['ansible-server']) {
         //building docker image starts
            sh "ssh -o StrictHostKeyChecking=no vagrant@${ansible_server_private_ip} cd /home/vagrant/"
            sh "ssh -o StrictHostKeyChecking=no vagrant@${ansible_server_private_ip} docker image build -t $JOB_NAME:v-$BUILD_ID ."
         //building docker image ends
         //Tagging docker image starts
            sh "ssh -o StrictHostKeyChecking=no vagrant@${ansible_server_private_ip} docker image tag $JOB_NAME:v-$BUILD_ID akramlh/$JOB_NAME:v-$BUILD_ID"
         sh "ssh -o StrictHostKeyChecking=no vagrant@${ansible_server_private_ip} docker image tag $JOB_NAME:v-$BUILD_ID akramlh/$JOB_NAME:latest"
         //Tagging docker image ends
        }
    }
    
    stage('push docker images to dockerhub'){
     sshagent(['ansible-server']) {
      withCredentials([string(credentialsId:'dockerhub_passwd', variable: 'dockerhub_passwd')]){
       sh "ssh -o StrictHostKeyChecking=no vagrant@${ansible_server_private_ip} docker login -u akramlh -p ${dockerhub_passwd}"
       sh "ssh -o StrictHostKeyChecking=no vagrant@${ansible_server_private_ip} docker image push akramlh/$JOB_NAME:v-$BUILD_ID"
       sh "ssh -o StrictHostKeyChecking=no vagrant@${ansible_server_private_ip} docker image push akramlh/$JOB_NAME:latest"
       
       //also delete old docker images
       sh "ssh -o StrictHostKeyChecking=no vagrant@${ansible_server_private_ip} docker image rm akramlh/$JOB_NAME:v-$BUILD_ID akramlh/$JOB_NAME:latest $JOB_NAME:v-$BUILD_ID"
      }
        }
    }
        stage('Clone Latest Code') {
                git branch: "${BRANCH}", url: "${REPO_URL}"
           
        }
    
 
stage('Clone or Update Repository on Ansible Server') {
    sshagent(['ansible-server']) {
        sh """
            ssh -o StrictHostKeyChecking=no vagrant@${ansible_server_private_ip} '
            # Navigate to the directory or create it if it doesn’t exist
            mkdir -p /home/akram/tp3-devops-k8s && cd /home/akram/tp3-devops-k8s

            # Clone the repository if it doesn't exist, otherwise pull the latest changes
            if [ ! -d .git ]; then
                git clone -b ${BRANCH} ${REPO_URL} .
            else
                git fetch origin
                git reset --hard origin/${BRANCH}
            fi
            '
        """
    }
}


 
}
