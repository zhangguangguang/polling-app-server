def label = "slave-${UUID.randomUUID().toString()}"

podTemplate(label: label, containers: [
  containerTemplate(name: 'maven', image: 'maven:3.6-alpine', command: 'cat', ttyEnabled: true),
  containerTemplate(name: 'docker', image: 'docker', command: 'cat', ttyEnabled: true),
  containerTemplate(name: 'kubectl', image: 'cnych/kubectl', command: 'cat', ttyEnabled: true),
  containerTemplate(name: 'helm', image: 'cnych/helm', command: 'cat', ttyEnabled: true)
], volumes: [
  hostPathVolume(mountPath: '/root/.m2', hostPath: '/var/run/m2'),
  hostPathVolume(mountPath: '/home/jenkins/.kube', hostPath: '/root/.kube'),
  hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock')
]) {
  node(label) {
    def myRepo = checkout scm
    def gitCommit = myRepo.GIT_COMMIT
    def gitBranch = myRepo.GIT_BRANCH
   
    stage('Prepare') {
        echo "1.Prepare Stage"
        script {
            build_tag = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
            if (env.BRANCH_NAME != 'master') {
                build_tag = "${env.BRANCH_NAME}-${build_tag}"
        }
      }
    }

    stage('单元测试') {
      echo "2.测试阶段"
    }

    stage('代码编译打包') {
      container('maven') {
        echo "3.打码编译打包阶段"
      }
    }

    stage('构建 Docker 镜像') {
      container('docker') {
        echo "4.构建 Docker 镜像阶段"
        sh "docker build -t polling-app-server:${build_tag} ."
      }
    }
   
    stage('Push') {
        echo "5.Push Docker Image Stage"
        withCredentials([usernamePassword(credentialsId: 'harbor', passwordVariable: 'DOCKER_HUB_PASSWORD', usernameVariable: 'DOCKER_HUB_USER')]) {
            sh "docker login ${dockerRegistryUrl} -u ${DOCKER_HUB_USER} -p ${DOCKER_HUB_PASSWORD}"
            sh "docker tag polling-app-server:${build_tag} harbor01.saicm.local/kubernetes-1-13-5/polling-app-server:${build_tag}"
            sh "docker push harbor01.saicm.local/kubernetes-1-13-5/polling-app-server:${build_tag}"
        }
    }

    stage('运行 Kubectl') {
      container('kubectl') {
        echo "6.查看 K8S 集群 Pod 列表"
        sh "kubectl get pods --all-namespaces" 
      }
    }
    stage('运行 Helm') {
      container('helm') {
        echo "7.查看 Helm Release 列表"
        sh "helm list"
      }
    }
  }
}
