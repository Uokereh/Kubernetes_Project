pipeline {

    agent any

    stage('Git Checkout'){
        git 'https://github.com/Uokereh/Kubernetes_Project.git'
    }

    stage('Send Docker file to Ansible Server over SSH'){
        sshagen(['ansible_demo']){
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.81.4' #YOUR-ANSIBLE-SERVER-IP-ADDRESS
            sh 'scp /var/lib/jenkins/workspace/pipeline-demo/* ubuntu@172.31.81.4:/home/ubuntu/' #YOUR-ANSIBLE-SERVER-IP-ADDRESS
        }
    }

    stage('Docker Build Image'){
        sshagent(['ansible_demo']){
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.81.4 cd /home/ubuntu/' #YOUR-ANSIBLE-SERVER-IP-ADDRESS
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.81.4 docker image build -t $JOB_NAME:v1.$BUILD_ID .' #YOUR-ANSIBLE-SERVER-IP-ADDRESS
        }
    }

    stage('Docker Image Tagging'){
        sshagent(['ansible_demo']){
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.81.4 cd /home/ubuntu/' #YOUR-ANSIBLE-SERVER-IP-ADDRESS
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.81.4 docker image tag $JOB_NAME:v1.$BUILD_ID uokereh/$JOB_NAME:v1.$BUILD_ID ' #YOUR-ANSIBLE-SERVER-IP-ADDRESS
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.81.4 docker image tag $JOB_NAME:v1.$BUILD_ID uokereh/$JOB_NAME:v1.$BUILD_ID ' #YOUR-ANSIBLE-SERVER-IP-ADDRESS

        }
    }

    stage('Push Docker Image to Docker Hub'){
        sshagent(['ansible_demo']){
            withCredentials([string(credentialsId: 'dockerhub_passwd', variable: 'dockerhub_passwd')]){
                sh "ssh -o StrictHostKeyChecking=no ubuntu@172.31.81.4 docker login -u uoekreh -p ${dockerhub_passwd}" #YOUR-ANSIBLE-SERVER-IP-ADDRESS
                sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.81.4 docker image push uokereh/$JOB_NAME:v1.$BUILD_ID ' #YOUR-ANSIBLE-SERVER-IP-ADDRESS
                sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.81.4 docker image push uokereh/$JOB_NAME:latest ' #YOUR-ANSIBLE-SERVER-IP-ADDRESS

                sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.81.4 docker image rm uokereh/$JOB_NAME:v1.$BUILD_ID uokereh/$JOB_NAME:latest $JOB_NAME:v1.$BUILD_ID' #YOUR-ANSIBLE-SERVER-IP-ADDRESS
            }

        }
    }

     stage('Copy Files From Ansible to Kubernetes Server'){
         sshagent(['kubernetes_server']){
             sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.19.112' #YOUR-KUBERNETES-SERVER-IP-ADDRESS
             sh 'scp /var/lib/jenkins/workspace/pipeline-demo/* ubuntu@172.31.19.112:/home/ubuntu/' #YOUR-KUBERNETES-SERVER-IP-ADDRESS
         }
     }

     stage('Kubernetes Deployment using Ansible'){
         sshagent(['kubernetes_server']){
             sh 'ssh -o StrictHostKeyChecking=node ubuntu@172.31.81.4 cd /home/ubuntu/' #YOUR-ANSIBLE-SERVER-IP-ADDRESS
             sh 'ssh -o StrictHostKeyChecking=node ubuntu@172.31.81.4 ansible-playbook ansible.yml' #YOUR-ANSIBLE-SERVER-IP-ADDRESS
         }
     }

}