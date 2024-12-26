@Library('Jenkins-Shared-library') _
def mvnCommand = env.mvncommand ?: 'install'
def deployEnv = env.DeploymentEnvironment ?: 'staging'
pipeline {
    agent none 

    stages {
        stage('Checkout Code') {
            agent { label 'Slave' } 
            steps { 
                echo "This is the Url from shared Library: ${constants.gitUrl}"
                echo "This is the password from shared Library: ${constants.password}"
                checkout([
                    $class: 'GitSCM', 
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                                                url: constants.gitUrl, 
                                                credentialsId: constants.password
                    ]]
                ])
            }
        }

        stage('Build') {
            agent { label 'Slave' } 
            steps {
                sh '''
                mvn clean ${mvncommand}
                '''
            }
        }

        stage('Test') {
            agent { label 'Slave1' } 
            steps {
                sh '''
                mvn test
                '''
            }
        }

        stage('Deploy to Artifactory') {
            agent { label 'Slave1' } 
            steps {
                configFileProvider([
                    configFile(fileId: constants.artifactorysetting, variable: 'MAVEN_SETTINGS')
                ]) {
                    sh '''
                    mvn deploy -s $MAVEN_SETTINGS
                    '''
                }
            }
        }
        stage("Deploy To Environment") {
            steps {
                    sh '''
                    echo "I am Deploying this war or application to ${deployEnv}"
                    '''

            }
        }
        // stage('Deploy to Tomcat') {
        //     agent { label 'Slave1' } 
        //     steps {
        //         sh '''
        //         cp -r /home/ubuntu/workspace/Pipeline_job/target/sample-webapp.war /home/ubuntu/tomcat/tomcat10/webapps
        //         sudo /home/ubuntu/tomcat/tomcat10/bin/shutdown.sh
        //         sudo /home/ubuntu/tomcat/tomcat10/bin/startup.sh
        //         '''
        //     }
        // }
    }
}
