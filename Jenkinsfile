pipeline {
agent {
        docker {
            image 'maven:3-alpine'
            args '-v /root/.m2:/root/.m2'
        }
    }
    stages {
        stage('Artifactory download and upload'){
            steps {
                script{
                    // Obtain an Artifactory server instance, defined in Jenkins --> Manage:
                    def server = Artifactory.server 'my-artifactory'

                    // Read the download and upload specs:
                    def uploadSpec = readFile 'resources/props-upload.json'

                    // Upload files to Artifactory:
                    def buildInfo1 = server.upload spec: uploadSpec

                    // Publish the merged build-info to Artifactory
                    server.publishBuildInfo buildInfo1
                }
            }
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
                rtUpload (
                    serverId: 'my-artifactory', // Obtain an Artifactory server instance, defined in Jenkins --> Manage:
                    spec: """{
                            "files": [
                                    {
                                        "pattern": "resources/*.jar",
                                        "target": "libs-snapshot-local"
                                    }
                                ]
                            }"""
                )
            }
        }
        stage('Deploy for production') {
            when {
                branch 'production'
            }
            steps {
                sh './jenkins/scripts/deploy-for-production.sh'
                input message: 'Finished testing production? (Click "Proceed" to continue)'
                rtUpload (
                    serverId: 'my-artifactory', // Obtain an Artifactory server instance, defined in Jenkins --> Manage:
                    spec: """{
                            "files": [
                                    {
                                        "pattern": "resources/*.jar",
                                        "target": "libs-release-local"
                                    }
                                ]
                            }"""
                )
            }
            }
    }
 }
