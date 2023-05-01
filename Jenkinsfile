pipeline {
  agent {
    kubernetes {
      yamlFile 'build-pod.yaml'  // path to the pod definition relative to the root of our project
    }
  }
  stages {
    stage('Build Docker Image') {
      steps {
        container('docker') {
          withCredentials([usernamePassword(credentialsId: 'harbor-pw', passwordVariable: 'password', usernameVariable: 'username')]) {
            sh """ 
              docker build -t 172.30.48.210/library/myweb:${BUILD_NUMBER} -f dockerfile .
              docker login http://172.30.48.210/ -p ${password} -u ${username}
              docker push 172.30.48.210/library/myweb:${BUILD_NUMBER}
              docker rmi -f 172.30.48.210/library/myweb:${BUILD_NUMBER} 172.30.48.210/library/httpd:1.0
            """  
          }
        }
      }
    }
    stage('deploy') {
      steps {
        container('jnlp') {
          configFileProvider([configFile(fileId: 'kube-admin-auth', targetLocation: 'kubeconfig')]) {
          sh """
          /home/jenkins/agent/workspace/Deploy/kubectl create deploy myweb${BUILD_NUMBER} -n myweb --image 172.30.48.210/library/myweb:${BUILD_NUMBER} --kubeconfig=kubeconfig
          """  
          }
        }
      }
    }
  }
}
