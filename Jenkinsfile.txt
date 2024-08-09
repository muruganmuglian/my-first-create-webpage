pipeline {
    agent any
    
    tools {
        maven 'local_maven'
    }
    parameters {
         string(name: 'staging_server', defaultValue: '13.232.37.20', description: 'Remote Staging Server')
    }

stages{
        stage('Build'){
            steps {
                sh 'mvn clean package'
            }
            post {
                success {
                    echo 'Archiving the artifacts'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

        stage ('Deployments'){
            parallel{
                stage ("Deploy to Staging"){
                    steps {
                         deploy adapters: [tomcat9(credentialsId: 'f4d1c554-4f42-4e06-b4be-6008060d1865', path: '', url: 'http://localhost:8080/manager')], contextPath: null, war: '**/*.war'
                    }
                }
            }
        }
    }
}
