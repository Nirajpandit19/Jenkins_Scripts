def registry = "https://triald31qcj.jfrog.io/artifactory"

pipeline {
    agent any
    environment {
        PATH = "/opt/maven/bin:$PATH"
    }

    stages {
        stage("Git Clone") {
            steps {
                git url: 'https://github.com/Nirajpandit19/java-hello-world-with-maven.git', branch: 'master'
            }
        }

        stage("Build & Deploy") {
            steps {
                echo "<--------Building Started--------->"
                withCredentials([usernamePassword(credentialsId: 'pandit', usernameVariable: 'ART_USER', passwordVariable: 'ART_PASS')]) {
                    sh """
                        mvn clean deploy \
                        -Dmaven.test.skip=true \
                        -DaltDeploymentRepository=artifactory::default::https://${ART_USER}:${ART_PASS}@${registry}/libs-release-local
                    """
                }
                echo "<--------Building Ended--------->"
            }
        }

        stage("Test") {
            steps {
                echo "<--------Testing Started--------->"
                sh 'mvn surefire-report:report'
                echo "<--------Testing Ended--------->"
            }
        }

        stage("Jar Publish") {
            steps {
                echo "<--------Jar Publish Started--------->"
                script {
                    withCredentials([usernamePassword(credentialsId: 'pandit', usernameVariable: 'ART_USER', passwordVariable: 'ART_PASS')]) {
                        def properties = "buildId=${env.BUILD_ID},commitid=${GIT_COMMIT}"
                        def uploadSpec = """{
                          "files": [
                            {
                              "pattern": "target/*.jar",
                              "target": "pandit-libs-release-local/",
                              "flat": true,
                              "props": "${properties}",
                              "exclusions": ["*.sha1","*.md5"]
                            }
                          ]
                        }"""

                        def server = Artifactory.newServer url: registry, credentialsId: "jfrog-creds"
                        def buildInfo = server.upload(uploadSpec)
                        buildInfo.env.collect()
                        server.publishBuildInfo(buildInfo)
                    }
                }
                echo "<--------Jar Publish Ended--------->"
            }
        }
    }
}
