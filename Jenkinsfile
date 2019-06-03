def helmPackage (path) {
    echo "Packaging ${path}"

    script {
        sh "helm package ${path}"
    }
}

def uploadChart (file) {
    echo "uploading chart to chartmuseum ${file}"

    script {
        sh 'curl --data-binary "@' + file + '" http://9.98.171.136:30552/api/charts'
    }
}

def helmInstall (chart, release) {
    echo "Installing ${chart} release ${release}"

    script {
        sh "helm del --purge ${release} --tls"
        sh "sleep 15"
        sh """
            helm upgrade --install ${release} ${chart} --tls --recreate-pods
        """
        
    }
}



node {
    def resultImage
    def voteImage
    def workerImage

    stage('Clone repository') {
        checkout scm
    }
    stage('build docker images') {
        resultImage = docker.build("roitev/result", "./src/result")
        voteImage = docker.build("roitev/vote", "./src/vote")
        workerImage = docker.build("roitev/worker", "./src/worker")

    }
    stage('test docker images') {
        resultImage.inside {
            sh 'echo result container'
        }

        voteImage.inside {
            sh 'echo vote container'
        }

        workerImage.inside {
            sh 'echo worker container'
        }
    }

    stage('Push docker images') {
        docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
            resultImage.push("${env.BUILD_NUMBER}")
            resultImage.push("latest")

            voteImage.push("${env.BUILD_NUMBER}")
            voteImage.push("latest")

            workerImage.push("${env.BUILD_NUMBER}")
            workerImage.push("latest")


        }
    }

    stage('cloudctl login') {
        script {
        sh "cloudctl login -a https://9.98.171.136:8443 --skip-ssl-validation -u admin -p admin -n voting"
        }
    }

    stage('helm init') {
        script {
        sh "helm init --client-only --skip-refresh"
        }
    }
    
    stage('tailor helm charts') {
        script {
        sh "sed -i -- 's/BUILD_VERSION/${env.BUILD_NUMBER}/g' ./helm_charts/*/*.yaml"
        sh "sed -i -- 's/BUILD_VERSION/${env.BUILD_NUMBER}/g' ./helm_charts/votingapplication/*/*.yaml"
        }
    }
    stage('helm package charts') {
        helmPackage('helm_charts/db')
        helmPackage('helm_charts/redis')
        helmPackage('helm_charts/result')
        helmPackage('helm_charts/vote')
        helmPackage('helm_charts/worker')
        helmPackage('helm_charts/votingapplication')

    }
    stage('upload helm charts to chartmuseum') {
        uploadChart("db-${env.BUILD_NUMBER}.tgz")
        uploadChart("redis-${env.BUILD_NUMBER}.tgz")
        uploadChart("result-${env.BUILD_NUMBER}.tgz")
        uploadChart("vote-${env.BUILD_NUMBER}.tgz")
        uploadChart("worker-${env.BUILD_NUMBER}.tgz")
    }

    stage('helm install votingapplication') {
        helmInstall('helm_charts/votingapplication', 'votingapp')
    }

}