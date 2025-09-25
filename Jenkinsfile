def registry = "https://triald31qcj.jfrog.io/artifactory"

pipeline {
    agent any
    
    tools {
        maven 'Maven-3.9.4'  // Name from Global Tool Configuration
        jdk 'Java-21'        // Name from Global Tool Configuration
    }
    
    environment {
        MAVEN_OPTS = '-Xmx1024m'
        JAVA_HOME = tool 'Java-21'
        MAVEN_HOME = tool 'Maven-3.9.4'
        PATH = "${MAVEN_HOME}/bin:${JAVA_HOME}/bin:${env.PATH}"
    }
    
    stages {
        stage("Environment Info") {
            steps {
                sh '''
                    echo "=== Environment Information ==="
                    echo "Java Version:"
                    java -version
                    echo "Java Compiler Version:"
                    javac -version
                    echo "Maven Version:"
                    mvn -version
                    echo "Environment Variables:"
                    echo "JAVA_HOME: $JAVA_HOME"
                    echo "MAVEN_HOME: $MAVEN_HOME"
                    echo "PATH: $PATH"
                    echo "================================"
                '''
            }
        }
        
        stage("Git Clone") {
            steps {
                git url: 'https://github.com/Nirajpandit19/java-hello-world-with-maven.git', branch: 'master'
            }
        }
        
        stage("Build") {
            steps {
                echo "<--------Building Started--------->"
                sh 'mvn clean compile'
                echo "<--------Building Ended--------->"
            }
        }
        
        stage("Test") {
            steps {
                echo "<--------Testing Started--------->"
                sh 'mvn test'
                sh 'mvn surefire-report:report'
                echo "<--------Testing Ended--------->"
            }
        }
        
        stage("Package") {
            steps {
                echo "<--------Packaging Started--------->"
                sh 'mvn package -DskipTests'
                echo "<--------Packaging Ended--------->"
            }
        }
        
        stage("Deploy to Artifactory") {
            steps {
                echo "<--------Deploy to Artifactory Started--------->"
                script {
                    try {
                        def server = Artifactory.newServer url: registry, credentialsId: "artifact_cred"
                        def buildInfo = Artifactory.newBuildInfo()
                        buildInfo.name = env.JOB_NAME
                        buildInfo.number = env.BUILD_NUMBER
                        buildInfo.env.collect()
                        
                        def uploadSpec = """{
  "files": [
    {
      "pattern": "target/*.jar",
      "target": "pandit-libs-release-local/",
      "flat": true,
      "props": "buildId=${env.BUILD_ID};commitId=${env.GIT_COMMIT};buildNumber=${env.BUILD_NUMBER};javaVersion=21",
      "exclusions": ["*.sha1", "*.md5"]
    }
  ]
}"""
                        
                        server.upload spec: uploadSpec, buildInfo: buildInfo
                        server.publishBuildInfo buildInfo
                        
                        echo "Artifacts successfully uploaded to Artifactory"
                    } catch (Exception e) {
                        echo "Error uploading to Artifactory: ${e.getMessage()}"
                        currentBuild.result = 'UNSTABLE'
                    }
                }
                echo "<--------Deploy to Artifactory Ended--------->"
            }
        }
    }
    
    post {
        always {
            script {
                try {
                    if (fileExists('target/surefire-reports/')) {
                        junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml'
                    }
                    if (fileExists('target/') && sh(script: "ls target/*.jar 2>/dev/null | wc -l", returnStdout: true).trim() != "0") {
                        archiveArtifacts artifacts: 'target/*.jar', allowEmptyArchive: true
                    }
                } catch (Exception e) {
                    echo "Warning: Post-build actions failed: ${e.getMessage()}"
                }
            }
        }
        success {
            echo 'Pipeline completed successfully with Java 21!'
        }
        failure {
            echo 'Pipeline failed!'
        }
        unstable {
            echo 'Pipeline completed with warnings.'
        }
    }
}
