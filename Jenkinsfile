pipeline {
    agent { label 'linux' }
    stages {
        
        stage('Build') {
             steps {
                 
                 sh 'mvn clean package'
                junit '**/target/surefire-reports/TEST-*.xml'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }
        stage("test") {
            sh 'mvn  test'
        }
        stage("Docker Login")
    {
        withCredentials([string(credentialsId: 'KHARBOR_PASSWORD', variable: 'PASSWORD')]) 
        {
            sh 'docker login https://kharbor.local -u test -p $PASSWORD'
        }
    } 
    stage("Docker build")
     {
        sh 'docker version'
        sh 'docker build -t jhooq-docker-demo .'
        sh 'docker image list'
        sh 'docker tag rahulwagh17/jhooq-docker-demo:jhooq-docker-demo  kharbor.local/cicd/jhooq-docker-demo:jhooq-docker-demo'
     }

    stage("Push Image to Docker Hub")
    {
        sh 'docker push  kharbor.local/cicd/jhooq-docker-demo:jhooq-docker-demo'
    }
    
       stage("Deploy") {
        def remote = [:]
        remote.name = 'K8S master'
        remote.host = '172.16.20.101'
        remote.user = 'root'
        remote.password = '1'
        remote.allowAnyHosts = true

        stage('Put k8s-spring-boot-deployment.yml onto k8smaster') {
            sshPut remote: remote, from: 'docker-compose.yml', into: '.'
        }

        stage('Deploy spring boot') {
          sshCommand remote: remote, command: "kubectl apply -f docker-compose.yml"
        }
    }
    }
}
