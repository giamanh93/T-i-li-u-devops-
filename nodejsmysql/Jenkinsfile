
pipeline {
  agent any
  options {
    buildDiscarder(logRotator(numToKeepStr: '5'))
  }
  environment {
    DOCKER_CREDENTIALS = credentials('nodejsapp')
    apinodejsTag2=sh(returnStdout: true, script: "git tag --sort version:refname | tail -1").trim()
    apinodejsTag1=sh(returnStdout: true, script: "basename  $gitlabBranch").trim()
  }
  stages {
    stage('Build') {
      steps {

        sh " git checkout $apinodejsTag1"  

        sh "printenv"
        echo "Current Tag version: apinodejsTag1= ${apinodejsTag1} "
        echo "Bigest Tag version: apinodejsTag2= ${env.apinodejsTag2} origin= $gitlabBranch"
        sh 'docker build -t registry.gitlab.com/xzhoang/nodejsmysql:$apinodejsTag1  . '
      }
    }
    stage('Login') {
      steps {
        
	echo "Docker Login"
        sh 'echo $DOCKER_CREDENTIALS_PSW | docker login -u $DOCKER_CREDENTIALS_USR registry.gitlab.com/xzhoang/nodejsmysql   --password-stdin'
      }
    }
    stage('Push') {
      steps {
        sh 'docker push registry.gitlab.com/xzhoang/nodejsmysql:$apinodejsTag1 '
      }
    }
    stage('Update TAG to env') {
      steps {
          script{
             def nodes = Jenkins.getInstance().getGlobalNodeProperties()
             def envVars= nodes.get(0).getEnvVars()
             envVars.put("apinodejsTag", "$apinodejsTag1")
             Jenkins.getInstance().save()
             }

        echo "Deploying Tag version: ${env.apinodejsTag} "
             }
                              }
  }
  post {
    
    always {
      sh 'docker logout'
    }
    
  }
}

