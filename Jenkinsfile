def registry = "https://triald31qcj.jfrog.io/artifactory"
def artifactoryServerId = "pandit-artifactory"

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
                    withCredentials([usernamePassword(credentialsId: 'artifact_cred', usernameVariable: 'ART_USER', passwordVariable: 'ART_PASS')]) {
                        // Create Maven settings.xml with server configuration
                        writeFile file: 'settings.xml', text: """<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
          http://maven.apache.org/xsd/settings-1.0.0.xsd">
    <servers>
        <server>
            <id>artifactory</id>
            <username>\${ART_USER}</username>
            <password>\${ART_PASS}</password>
        </server>
    </servers>
</settings>"""
                        
                        // Deploy using the custom settings.xml
                        sh """
                            mvn deploy \\
                            -DskipTests \\
                            -s settings.xml \\
                            -DaltDeploymentRepository=artifactory::default::${registry}/pandit-libs-release-local
                        """
                    }
                }
                echo "<--------Deploy to Artifactory Ended--------->"
            }
        }
        
        stage("Jar Publish with Build Info") {
            steps {
                echo "<--------Jar Publish Started--------->"
                script {
                    try {
                        // Configure Artifactory server
                        def server = Artifactory.newServer url: registry, credentialsId: "artifact_cred"
                        
                        // Create upload spec with proper JSON formatting
                        def uploadSpec = """{
  "files": [
    {
      "pattern": "target/*.jar",
      "target": "pandit-libs-release-local/",
      "flat": true,
      "props": "buildId=${env.BUILD_ID};commitId=${env.GIT_COMMIT};buildNumber=${env.BUILD_NUMBER}",
      "exclusions": ["*.sha1", "*.md5"]
    }
  ]
}"""
                        
                        // Upload artifacts and collect build info
                        def buildInfo = Artifactory.newBuildInfo()
                        buildInfo.name = env.JOB_NAME
                        buildInfo.number = env.BUILD_NUMBER
                        buildInfo.env.collect()
                        
                        server.upload spec: uploadSpec, buildInfo: buildInfo
                        server.publishBuildInfo buildInfo
                        
                        echo "Artifacts uploaded and build info published successfully"
                    } catch (Exception e) {
                        echo "Warning: Jar publish failed: ${e.getMessage()}"
                        echo "Continuing with pipeline execution..."
                        currentBuild.result = 'UNSTABLE'
                    }
                }
                echo "<--------Jar Publish Ended--------->"
            }
        }
    }
    
    post {
        always {
            // Archive test results if they exist
            script {
                try {
                    if (fileExists('target/surefire-reports/')) {
                        junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml'
                    } else {
                        echo "No test results found"
                    }
                } catch (Exception e) {
                    echo "Warning: Could not archive test results: ${e.getMessage()}"
                }
            }
            
            // Archive artifacts if they exist
            script {
                try {
                    if (fileExists('target/') && sh(script: "ls target/*.jar 2>/dev/null | wc -l", returnStdout: true).trim() != "0") {
                        archiveArtifacts artifacts: 'target/*.jar', allowEmptyArchive: true
                    } else {
                        echo "No JAR artifacts found to archive"
                    }
                } catch (Exception e) {
                    echo "Warning: Could not archive artifacts: ${e.getMessage()}"
                }
            }
            
            // Clean temporary files
            script {
                try {
                    if (fileExists('settings.xml')) {
                        sh 'rm -f settings.xml'
                    }
                } catch (Exception e) {
                    echo "Warning: Could not clean temporary files: ${e.getMessage()}"
                }
            }
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed! Check the logs for more details.'
        }
        unstable {
            echo 'Pipeline completed with warnings - some stages may have failed but pipeline continued.'
        }
    }
}
