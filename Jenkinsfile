pipeline {
    agent none // No default agent; specify agents per stage

    stages {
        stage('Checkout Code') {
            agent { label 'Slave' } // Specify the server for SCM
            steps {
                checkout([
                    $class: 'GitSCM', 
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/Sumukha97/sample-webapp.git', 
                        credentialsId: 'github'
                    ]]
                ])
            }
        }

        stage('Build') {
            agent { label 'Slave1' } // Specify the server for Maven build
            steps {
                sh '''
                mvn clean install
                '''
            }
        }

        stage('Test') {
            agent { label 'Slave1' } // Reuse the same server for testing
            steps {
                sh '''
                mvn test
                '''
            }
        }

        stage('Deploy to Artifactory') {
            agent { label 'Slave1' } // Reuse the same server for deployment
            steps {
                configFileProvider([
                    configFile(fileId: 'f908b2e4-7249-426e-8b20-ab7bb953e1c2', variable: 'MAVEN_SETTINGS')
                ]) {
                    sh '''
                    mvn deploy -s $MAVEN_SETTINGS
                    '''
                }
            }
        }

        stage('Deploy to Tomcat') {
            agent { label 'Slave1' } // Specify the server for Tomcat deployment
            steps {
                sh '''
                cp -r /home/ubuntu/workspace/Pipeline_job/target/sample-webapp.war /home/ubuntu/tomcat/tomcat10/webapps
                sudo /home/ubuntu/tomcat/tomcat10/bin/shutdown.sh
                sudo /home/ubuntu/tomcat/tomcat10/bin/startup.sh
                '''
            }
        }
    }
}
