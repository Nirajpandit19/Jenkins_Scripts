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
                    withCredentials([usernamePassword(credentialsId: 'pandit', usernameVariable: 'ART_USER', passwordVariable: 'ART_PASS')]) {
                        // Configure Maven settings for deployment
                        sh """
                            mvn deploy \
                            -DskipTests \
                            -DaltDeploymentRepository=artifactory::default::${registry}/pandit-libs-release-local \
                            -Dusername=${ART_USER} \
                            -Dpassword=${ART_PASS}
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
                    // Configure Artifactory server
                    def server = Artifactory.newServer url: registry, credentialsId: "pandit"
                    
                    // Create upload spec
                    def uploadSpec = """{
                      "files": [
                        {
                          "pattern": "target/*.jar",
                          "target": "pandit-libs-release-local/",
                          "flat": true,
                          "props": "buildId=${env.BUILD_ID};commitId=${env.GIT_COMMIT}",
                          "exclusions": ["*.sha1", "*.md5"]
                        }
                      ]
                    }"""
                    
                    // Upload artifacts and collect build info
                    def buildInfo = Artifactory.newBuildInfo()
                    buildInfo.env.collect()
                    server.upload spec: uploadSpec, buildInfo: buildInfo
                    server.publishBuildInfo buildInfo
                }
                echo "<--------Jar Publish Ended--------->"
            }
        }
    }
    
    post {
        always {
            // Archive test results
            junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml'
            
            // Archive artifacts
            archiveArtifacts artifacts: 'target/*.jar', allowEmptyArchive: true
            
            // Clean workspace
            cleanWs()
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
