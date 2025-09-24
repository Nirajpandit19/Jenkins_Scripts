def registry = "https://triald31qcj.jfrog.io"

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

        stage("Build") {
            steps {
                echo "<--------Building Started--------->"
                sh 'mvn clean deploy -Dmaven.test.skip=true'
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
                script {
                    echo "<--------Jar Publish Started--------->"

                    def server = Artifactory.newServer(
                        url: registry + "/artifactory",
                        credentialsId: "38c3d30d-602b-45bd-98aa-76de50fcc84d"
                    )

                    def properties = "buildId=${env.BUILD_ID};commitid=${GIT_COMMIT}"

                    def uploadSpec = """{
                        "files": [
                            {
                                "pattern": "jarstaging/(*)",
                                "target": "pandit-libs-release-local",
                                "flat": false,
                                "props": "${properties}",
                                "exclusions": ["*.sha1","*.md5"]
                            }
                        ]
                    }"""

                    def buildInfo = server.upload(uploadSpec)
                    buildInfo.env.collect()
                    server.publishBuildInfo(buildInfo)

                    echo "<--------Jar Publish Ended--------->"
                }
            }
        }
    }
}

