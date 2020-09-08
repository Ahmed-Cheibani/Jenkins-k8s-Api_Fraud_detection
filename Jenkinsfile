podTemplate(
    label: 'mypod', 
    inheritFrom: 'default',
    containers: [
        containerTemplate(
            name: 'docker', 
            image: 'docker:18.02',
            ttyEnabled: true,
            command: 'cat'
        ),
        containerTemplate(
            name: 'helmmm', 
            image: 'ahmedcheibani/jenkins-slave-kubectl-helm:latest',
            ttyEnabled: true,
            command: 'cat'
        )
    ],
    volumes: [
        hostPathVolume(
            hostPath: '/var/run/docker.sock',
            mountPath: '/var/run/docker.sock'
        )
    ]
) {
    node('mypod') {
        def commitId
        stage ('Checkout Source') {
            checkout scm
            commitId = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
            sh "echo $commitId"
        }
        }
        def repository
        stage ('Docker build && push ') {
            container ('docker') {
                def registryIp = sh(script: 'getent hosts registry.kube-system | awk \'{ print $1 ; exit }\'', returnStdout: true).trim()
                sh "echo $registryIp"
                repository = "ahmedcheibani/fraudapp"
                withCredentials([usernamePassword(credentialsId: 'dockerhub_login', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')])
                {
                sh 'docker login --username="${USERNAME}" --password="${PASSWORD}"'
                sh "docker build -t ${repository}:${commitId} ."
                sh "docker push ${repository}:${commitId}"
                }
            }
        }
        stage ('Helm Deploy') {
            container ('helmmm') {
                sh "helm upgrade apifraud fraudapp-chart -n fraud -i --wait --set image.repository=${repository},image.tag=${commitId}"
            }
        }
    }
}