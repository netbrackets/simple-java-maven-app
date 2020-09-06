pipeline {
  agent {
    docker {
      image 'maven:3-alpine'
      args '-v /root/.m2:/root/.m2'
    }

  }
  stages {
    stage('Initialize') {
      steps {
        sh '''echo PATH = ${PATH}
echo M2_HOME = ${M2_HOME}
mvn -B -DskipTests clean package'''
      }
    }

    stage('Build') {
      post {
        always {
          junit 'target/surefire-reports/*.xml'
        }

      }
      steps {
        sh 'mvn -Dmaven.test.failure.ignore=true install'
      }
    }

    stage('Report') {
      when {
        branch 'master'
      }
      steps {
        junit 'target/surefire-reports/** /*.xml'
        archiveArtifacts 'target/*.jar,target/*.hpi'
      }
    }

    stage('Deliver for development') {
      when {
        branch 'development'
      }
      steps {
        sh './jenkins/scripts/deliver-for-development.sh'
        input 'Finished testing development? (Click "Proceed" to continue)'
      }
    }

    stage('Deploy for production') {
      when {
        branch 'production'
      }
      steps {
        sh './jenkins/scripts/deploy-for-production.sh'
        input 'Finished testing production? (Click "Proceed" to continue)'
      }
    }

  }
}