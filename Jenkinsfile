pipeline {
    agent {
        node {
            label 'docker'
        }
    }
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build docker image') {
            when {
               branch 'master'
        }
            steps {
                script{
                    app = docker.build("mydocker15/train")
                    app.inside {
                     sh 'echo $(curl localhost:8080)'    
                    }
                    
                }
            }
        }
        stage('Push docker image'){
            when {
                branch 'master'
        }
            steps {
                script{
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login'){
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('Deploy to Production'){
          when {
                branch 'master'
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker',usernameVariable: 'USERNAME',passwordVariable: 'USERPASS')]){
                    script{
                       sh "sshpass -p $USERPASS -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker pull mydocker15/train:${env.BUILD_NUMBER}\""
                        try{
                          sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker stop train\""
                          sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker rm train\""   
                        }
                        catch(err){
                         echo: 'caught error: $err'   
                        }
                         sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker run --restart always --name train -p 80:8080 -d mydocker15/train:${env.BUILD_NUMBER}\""
                    }
                }
            }
            
        }
                
    }
}
