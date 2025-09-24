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
                sh """
                    mvn clean deploy \
                    -Dmaven.test.skip=true \
                    -DaltDeploymentRepository=artifactory::default::${registry}/libs-release-local
                """
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
    }
}
