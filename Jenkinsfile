library identifier: 'jenkins-shared@master', retriever: modernSCM(
 [$class: 'GitSCMSource',
  remote: 'https://github.com/MobodidTech/jenkins-shared.git',
 ])

pipeline {
 environment {
  appName = "server"
  registry = "jdk2588/ci-cd"
  registryCredential = "docker"
  projectPath = "/jenkins/data/workspace/django-server"
 }

 agent any

 parameters {
  gitParameter name: 'RELEASE_TAG',
   type: 'PT_TAG',
   defaultValue: 'master'
 }

 stages {

  stage('Basic Information') {
   steps {
    sh "echo tag: ${params.RELEASE_TAG}"
   }
  }

  stage('Check Lint') {
   steps {
    sh "docker run --rm $registry:${params.RELEASE_TAG} flake8 --ignore=E501,F401,W391"
   }
  }

  stage('Run Tests') {
   steps {
    sh "docker run -v $projectPath/reports:/app/reports  --rm --network='host' $registry:${params.RELEASE_TAG} python martor_demo/manage.py test"
   }
  }

  stage('Build Image') {
   steps {
    script {
     if (isMaster()) {
      dockerImage = docker.build "$registry:master"
     }
    }
   }
  }

  stage('Push Image') {
   steps {
    script {
     if (isMaster()) {
      docker.withRegistry("", registryCredential) {
      dockerImage.push()
      } 
     }
    }
   }
  }

  stage('Notify Telegram') {
   steps() {
    script {
     if (isMaster()) {
      telegram.sendTelegram("Build successful for ${getBuildName()}\n" +
      "image $registry:${params.RELEASE_TAG} is pushed to DockerHub and ready to be deployed")
     }
    }
   }
  }

  stage('Garbage Collection') {
   steps {
    if (isMaster()) {
       sh "docker rmi $registry:${params.RELEASE_TAG}"
    }
   }
  }
 }

 post {
  failure {
   script {
    telegram.sendTelegram("Build failed for ${getBuildName()}\n" +
     "Checkout Jenkins console for more information. If you are not a developer simply ignore this message.")
   }
  }
 }

}

def getBuildName() {
 "${BUILD_NUMBER}_$appName:${params.RELEASE_TAG}"
}

def isMaster() {
 "${params.RELEASE_TAG}" == "master"
}
