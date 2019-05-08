def label = "slave-${UUID.randomUUID().toString()}"

podTemplate(label: label, name: 'jnlp', namespace: 'kube-ops',   containers: [
  containerTemplate(name: 'jnlp', image: 'cnych/jenkins:jnlp', diectory: '/home/jenkins', namespace: 'kube-ops')
], volumes: [
  hostPathVolume(mountPath: '/home/jenkins/.kube', hostPath: '/root/.kube'),
  hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock')
])
node('lable') {
    stage('Prepare') {
        echo "1.Prepare Stage"
        checkout scm
        script {
            build_head = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
            build_date = sh(returnStdout: true, script: 'date +%Y%m%d%H%M%S').trim()
            build_tag = "${build_date}-${build_head}"
            if (env.BRANCH_NAME != 'master') {
                build_tag = "${env.BRANCH_NAME}-${build_tag}"
            }
        }
    }
    stage('Test') {
      echo "2.Test Stage"
    }
    stage('Build') {
        echo "3.Build Docker Image Stage"
        sh "docker build -t registry.cn-shenzhen.aliyuncs.com/e6yun/devops-test:${build_tag} ."
    }
    stage('Push') {
        echo "4.Push Docker Image Stage"
        withCredentials([usernamePassword(credentialsId: 'alihub', passwordVariable: 'alihubPassword', usernameVariable: 'alihubUser')]) {
            sh "docker login -u ${alihubUser} -p ${alihubPassword} registry.cn-shenzhen.aliyuncs.com"
            sh "docker push registry.cn-shenzhen.aliyuncs.com/e6yun/devops-test:${build_tag}"
        }
    }
    stage('Deploy') {
        echo "5. Deploy Stage"
        if (env.BRANCH_NAME == 'master') {
            input "确认要部署线上环境吗？"
        }
        sh "sed -i 's/<BUILD_TAG>/${build_tag}/' k8s.yaml"
        sh "sed -i 's/<BRANCH_NAME>/${env.BRANCH_NAME}/' k8s.yaml"
        sh "kubectl apply -f k8s.yaml --record"
    }
}
