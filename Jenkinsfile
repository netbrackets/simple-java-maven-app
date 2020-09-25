def uploadSpec = """{
  "files": [
    {
      "pattern": "*.jar",
      "target": "bazinga-repo/froggy-files/"
    }
 ]
}"""
server.upload(uploadSpec)
pipeline {
agent {
        docker {
            image 'maven:3-alpine'
            args '-v /root/.m2:/root/.m2'
        }
    }
    stages {
        stage ('Artifactory configuration') {
          // Obtain an Artifactory server instance, defined in Jenkins --> Manage:
          server = Artifactory.server 'my-artifactory'

          rtMaven = Artifactory.newMavenBuild()
          rtMaven.tool = MAVEN_TOOL // Tool name from Jenkins configuration
          rtMaven.deployer releaseRepo: 'libs-release-local', snapshotRepo: 'libs-snapshot-local', server: server
          rtMaven.resolver releaseRepo: 'libs-release', snapshotRepo: 'libs-snapshot', server: server
          rtMaven.deployer.deployArtifacts = false // Disable artifacts deployment during Maven run

          buildInfo = Artifactory.newBuildInfo()
        }

        stage('Build') {
            steps {
                sh 'mvn -B -DskipTests clean package'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        stage('Deliver') {
            when {
                branch 'master'
            }
            steps {
                sh './jenkins/scripts/deliver.sh'
            }
        }
        stage('Deliver for development') {
            when {
                branch 'development'
            }
            steps {
                sh './jenkins/scripts/deliver-for-development.sh'
                input message: 'Finished testing development? (Click "Proceed" to continue)'
            }
        }
        stage('Deploy for production') {
            when {
                branch 'production'
            }
            steps {
                sh './jenkins/scripts/deploy-for-production.sh'
                input message: 'Finished testing production? (Click "Proceed" to continue)'
            }
    }
 }
 }
