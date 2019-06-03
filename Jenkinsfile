def helmPackage (path) {
    echo "Packaging ${path}"

    script {
        sh "helm package ${path}"
    }
}

def uploadChart (file) {
    echo "uploading chart to chartmuseum ${file}"

    script {
        sh "curl --data-binary "\@$(file)" http://9.98.171.136:30552/api/charts"
    }
}


node {
    def resultImage
    def voteImage
    def workerImage

    stage('Clone repository') {
        checkout scm
    }
    stage('build result docker image') {
        resultImage = docker.build("roitev/result", "./src/result") 
    }
    stage('test result image') {
        resultImage.inside {
            sh 'echo result container'
        }
    }
    stage('Push result image') {
        docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
            resultImage.push("${env.BUILD_NUMBER}")
            resultImage.push("latest")
        }
    }

    stage('build vote docker image') {
        voteImage = docker.build("roitev/vote", "./src/vote") 
    }
    stage('test vote image') {
        voteImage.inside {
            sh 'echo vote container'
        }
    }
    stage('Push vote image') {
        docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
            voteImage.push("${env.BUILD_NUMBER}")
            voteImage.push("latest")
        }
    }

    stage('build worker docker image') {
        workerImage = docker.build("roitev/worker", "./src/worker") 
    }
    stage('test worker image') {
        workerImage.inside {
            sh 'echo worker container'
        }
    }
    stage('Push worker image') {
        docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
            workerImage.push("${env.BUILD_NUMBER}")
            workerImage.push("latest")
        }
    }

    stage('cloudctl login') {
        script {
        sh "cloudctl login -a https://9.98.171.136:8443 --skip-ssl-validation -u admin -p admin -n default"
        }
    }
    stage('helm init') {
        script {
        sh "helm init --client-only --skip-refresh"
        }
    }
    
    stage('helm package db') {
        helmPackage('helm_charts/db')
    }
    stage('upload helm chart db') {
        uploadChart('db-0.1.0.tgz')
    }

    


}