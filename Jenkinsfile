ansible_server_private_ip="192.168.0.14"
kubernetes_server_private_ip="192.168.0.12"
node{
    stage('Git checkout'){
        //replace with your github repo url
        git branch: 'main', url: 'https://github.com/akramLh005/TP3-DevOps-k8s.git'
    }
    
     //all below sshagent variables created using Pipeline syntax
stage('Sending Dockerfile to Ansible server') {
    sshagent(['ansible-server']) {
        // Disable pseudo-terminal allocation with -T and test connection
        sh "ssh -o StrictHostKeyChecking=no vagrant@${ansible_server_private_ip} 'echo Connection established'"

        // Use scp with the same credentials
        sh "scp -o StrictHostKeyChecking=no /var/lib/jenkins/workspace/devops-project-one/* vagrant@${ansible_server_private_ip}:/home/vagrant"
    }
}

    
    stage('Docker build image'){
        sshagent(['ansible-server']) {
         //building docker image starts
            sh "ssh -o StrictHostKeyChecking=no vagrant@${ansible_server_private_ip} cd /home/vagrant/"
            sh "ssh -o StrictHostKeyChecking=no vagrant@${ansible_server_private_ip} docker image build -t $JOB_NAME:v-$BUILD_ID ."
         //building docker image ends
         //Tagging docker image starts
            sh "ssh -o StrictHostKeyChecking=no vagrant@${ansible_server_private_ip} docker image tag $JOB_NAME:v-$BUILD_ID kubemubin/$JOB_NAME:v-$BUILD_ID"
         sh "ssh -o StrictHostKeyChecking=no vagrant@${ansible_server_private_ip} docker image tag $JOB_NAME:v-$BUILD_ID kubemubin/$JOB_NAME:latest"
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
    
    stage('Copy files from jenkins to kubernetes server'){
     sshagent(['kubernetes-server']) {
      sh "ssh -o StrictHostKeyChecking=no ubuntu@${kubernetes_server_private_ip} cd /home/ubuntu/"
      sh "scp /var/lib/jenkins/workspace/devops-project-one/* ubuntu@${kubernetes_server_private_ip}:/home/ubuntu"
     }
    }
 
    stage('Kubernetes deployment using ansible'){
     sshagent(['ansible-server']) {
      sh "ssh -o StrictHostKeyChecking=no vagrant@${ansible_server_private_ip} cd /home/vagrant/"
      sh "ssh -o StrictHostKeyChecking=no vagrant@${ansible_server_private_ip} ansible -m ping ${kubernetes_server_private_ip}"
      sh "ssh -o StrictHostKeyChecking=no vagrant@${ansible_server_private_ip} ansible-playbook ansible-playbook.yml"
     } 
    }
 
}
