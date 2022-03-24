node {

    stage('Git Checkout'){
        git 'https://github.com/Uokereh/Kubernetes_Project.git'
    }

    stage('Send Docker file to Ansible Server over SSH'){
        sshagent(['ansible_demo']){
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.81.4' 
            sh 'scp /var/lib/jenkins/workspace/pipeline-demo-1/* ubuntu@172.31.81.4:/home/ubuntu/' 
        }
    }

    stage('Docker Build Image'){
        sshagent(['ansible_demo']){
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.81.4 cd /home/ubuntu/' 
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.81.4 docker image build -t $JOB_NAME:v1.$BUILD_ID .' 
        }
    }

    stage('Docker Image Tagging'){
        sshagent(['ansible_demo']){
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.81.4 cd /home/ubuntu/' 
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.81.4 docker image tag $JOB_NAME:v1.$BUILD_ID uokereh/$JOB_NAME:v1.$BUILD_ID ' 
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.81.4 docker image tag $JOB_NAME:v1.$BUILD_ID uokereh/$JOB_NAME:v1.$BUILD_ID ' 

        }
    }

    stage('Push Docker Image to Docker Hub'){
        sshagent(['ansible_demo']){
            withCredentials([string(credentialsId: 'dockerhub_passwd', variable: 'dockerhub_passwd')]){
                sh "ssh -o StrictHostKeyChecking=no ubuntu@172.31.81.4 docker login -u uokereh -p ${dockerhub_passwd}" 
                sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.81.4 docker image push uokereh/$JOB_NAME:v1.$BUILD_ID ' 
                sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.81.4 docker image push uokereh/$JOB_NAME:latest ' 

                sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.81.4 docker image rm uokereh/$JOB_NAME:v1.$BUILD_ID uokereh/$JOB_NAME:latest $JOB_NAME:v1.$BUILD_ID' 
            }

        }
    }

     stage('Copy Files From Ansible to Kubernetes Server'){
         sshagent(['kubernetes_server']){
             sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.19.112' 
             sh 'scp /var/lib/jenkins/workspace/pipeline-demo-1/* ubuntu@172.31.19.112:/home/ubuntu/' 
         }
     }

     stage('Kubernetes Deployment using Ansible'){
         sshagent(['ansible_demo']){
             sh 'ssh -o StrictHostKeyChecking=node ubuntu@172.31.81.4 cd /home/ubuntu/' 
             sh 'ssh -o StrictHostKeyChecking=node ubuntu@172.31.81.4 ansible-playbook ansible.yml' 
         }
     }

}