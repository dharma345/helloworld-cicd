def label = "jenkins-slave-${UUID.randomUUID().toString()}"
podTemplate(label: label, containers: [
    containerTemplate(name: 'slave', image: 'durgaprasad444/jenmine:1.1', ttyEnabled: true, command: 'cat')
],
volumes: [
  hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock')
]) {
    node(label) {
        def APP_NAME = "helloworld-cicd"
        def tag = "dev"
            stage("clone code") {
                container('slave') {
                    
                    // Let's clone the source
                    sh """ 
                      git clone https://github.com/durgaprasad444/${APP_NAME}.git            
                      cd ${APP_NAME}
                      cp -rf * /home/jenkins/workspace/maven-example
                    """
                }
            }
        stage("mvn build") {
            container('slave') {
                    // If you are using Windows then you should use "bat" step
                    // Since unit testing is out of the scope we skip them
                    sh "mvn clean install"
            }
        }
        

        stage('Build image') {
            container('slave') {
                sh """
                cd /home/jenkins/workspace/maven-example
                docker build -t durgaprasad444/${APP_NAME}-${tag}:$BUILD_NUMBER .
                """
                
  
}
}
    stage('PUSH IMAGE') {
                 container('slave') {
                 withCredentials([[$class: 'UsernamePasswordMultiBinding',
                credentialsId: 'dockerhub',
                usernameVariable: 'DOCKER_HUB_USER',
                passwordVariable: 'DOCKER_HUB_PASSWORD']]) {
                 sh "docker login -u ${DOCKER_HUB_USER} -p ${DOCKER_HUB_PASSWORD}"
                 sh "docker push durgaprasad444/${APP_NAME}-${tag}:$BUILD_NUMBER"
    
    
            }
             }
            }
        
        
        
        stage("deploy on kubernetes") {
            container('slave') {
                sh "kubectl apply -f hello-kubernetes.yaml"
                sh "kubectl set image deployment/hello-kubernetes hello-kubernetes=gcr.io/sentrifugo/${APP_NAME}-${tag}:$BUILD_NUMBER"
            }
        }
                }
            }
