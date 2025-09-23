pipeline{
    agent any
        environment {
            PATH = "/opt/maven/bin:$PATH"
        }
        stages {
            stage("git clone"){
                steps {
                    git url: 'https://github.com/Nirajpandit19/java-hello-world-with-maven.git' , branch: 'master'
                }
            }
            stage("Building"){
                steps{
                    sh 'mvn clean install'
                }
            }
        }
}
