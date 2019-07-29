podTemplate(label: 'demo-service',serviceAccount: 'my-jenkins',containers: [
  containerTemplate(name: 'gradle', image: 'gradle:5.5.1-jre8', command: 'cat', ttyEnabled: true),
  containerTemplate(name: 'docker', image: 'docker', command: 'cat', ttyEnabled: true),
  containerTemplate(name: 'helm', image: 'lachlanevenson/k8s-helm:latest', command: 'cat', ttyEnabled: true)
],
volumes: [
  hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock')
]) {
  node('demo-service') {
      stage('Build Source Code to Jar file') {
           git url: 'https://github.com/wiselyman/demo-service.git', credentialsId: 'github', branch: 'master'
           container('gradle') {
              sh "gradle bootJar"
              def projectVersion = sh script: "gradle properties -q | grep "version:" | awk '{print $2}'", returnStdout: true
              env.version = projectVersion
             }
       }

      stage('Build Docker Image and Push to Docker Registry') {
            container('docker'){
                 docker.withRegistry("http://registry.cn-hangzhou.aliyuncs.com", "aliyun"){
                   docker.build("wiselyman/demo-service:${env.version}").push(env.version)
                 }
            }
       }

       stage('Deploy to Kubernetes Cluster'){
            container('kubectl'){
               sh "helm upgrade --install --force --set image.tag=${env.version} demo-service demo-service/"
       }

    }
}