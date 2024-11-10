ansible_server_private_ip="192.168.0.14"
kubernetes_server_private_ip="192.168.0.12"
REPO_URL="https://github.com/akramLh005/tp3-devops-k8s.git"
BRANCH= "main"   

node {
    stage('Git Checkout') {
        // Pull the latest code from the specified branch
        git branch: "${BRANCH}", url: "${REPO_URL}"
    }

    stage('Sending Dockerfile to Ansible Server') {
        sshagent(['ansible-server']) {
            // Test SSH connection to the Ansible server
            sh "ssh -o StrictHostKeyChecking=no vagrant@${ansible_server_private_ip} 'echo Connection established'"

            // Clean up the target directory on Ansible server
            sh "ssh -o StrictHostKeyChecking=no vagrant@${ansible_server_private_ip} 'rm -rf /home/vagrant/tp3-devops-k8s/*'"

            // Use scp to transfer all files from Jenkins workspace to Ansible server
            sh "scp -o StrictHostKeyChecking=no ${env.WORKSPACE}/* vagrant@${ansible_server_private_ip}:/home/vagrant/tp3-devops-k8s/"
        }
    }

    stage('Docker build image') {
        sshagent(['ansible-server']) {
            // Building Docker image
            sh "ssh -o StrictHostKeyChecking=no vagrant@${ansible_server_private_ip} 'cd /home/vagrant/tp3-devops-k8s && docker image build -t $JOB_NAME:v-$BUILD_ID .'"

            // Tagging Docker image
            sh "ssh -o StrictHostKeyChecking=no vagrant@${ansible_server_private_ip} 'docker image tag $JOB_NAME:v-$BUILD_ID akramlh/$JOB_NAME:v-$BUILD_ID'"
            sh "ssh -o StrictHostKeyChecking=no vagrant@${ansible_server_private_ip} 'docker image tag $JOB_NAME:v-$BUILD_ID akramlh/$JOB_NAME:latest'"
        }
    }

    stage('Push Docker Images to DockerHub') {
        sshagent(['ansible-server']) {
            withCredentials([string(credentialsId: 'dockerhub_passwd', variable: 'dockerhub_passwd')]) {
                sh "ssh -o StrictHostKeyChecking=no vagrant@${ansible_server_private_ip} 'docker login -u akramlh -p ${dockerhub_passwd}'"
                sh "ssh -o StrictHostKeyChecking=no vagrant@${ansible_server_private_ip} 'docker image push akramlh/$JOB_NAME:v-$BUILD_ID'"
                sh "ssh -o StrictHostKeyChecking=no vagrant@${ansible_server_private_ip} 'docker image push akramlh/$JOB_NAME:latest'"

                // Also delete old Docker images
                sh "ssh -o StrictHostKeyChecking=no vagrant@${ansible_server_private_ip} 'docker image rm akramlh/$JOB_NAME:v-$BUILD_ID akramlh/$JOB_NAME:latest $JOB_NAME:v-$BUILD_ID'"
            }
        }
    }

    stage('Kubernetes Deployment Using Ansible') {
        sshagent(['ansible-server']) {
            sh """
                ssh -o StrictHostKeyChecking=no vagrant@${ansible_server_private_ip} '
                mkdir -p /home/vagrant/tp3-devops-k8s && cd /home/vagrant/tp3-devops-k8s

                # Clone the repository if it doesn’t exist, otherwise pull the latest changes
                if [ ! -d .git ]; then
                    git clone -b ${BRANCH} ${REPO_URL} .
                else
                    git fetch origin
                    git reset --hard origin/${BRANCH}
                fi

                # Test Ansible connection and run the playbook
                cd /home/vagrant/tp3-devops-k8s &&
                ansible ws1 -m ping  &&
                ansible-playbook /home/vagrant/tp3-devops-k8s/ansible-playbook.yml
                '
            """
        }
    }

    stage('Kubernetes Port-Forwarding') {
        sh """
        kubectl port-forward --address 0.0.0.0 svc/myfirstdevopsservice 30000:80
        """
    }

    stage('Deploy Prometheus and Grafana Monitoring') {
        sshagent(['ansible-server']) {
            sh """
            ssh -o StrictHostKeyChecking=no vagrant@${ansible_server_private_ip} '
            # Install Helm if not already installed
            if ! command -v helm &> /dev/null; then
                curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
            fi

            # Add Prometheus and Grafana Helm repositories
            helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
            helm repo add grafana https://grafana.github.io/helm-charts
            helm repo update

            # Install Prometheus and Grafana using kube-prometheus-stack Helm chart
            helm install prometheus prometheus-community/kube-prometheus-stack --namespace monitoring --create-namespace

            # Port-forwarding Grafana (Optional, to access from localhost:3000)
            nohup kubectl port-forward -n monitoring svc/prometheus-grafana 3000:80 > /dev/null 2>&1 &
            '
            """
        }
    }
}
