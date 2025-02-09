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

        stage('SonarQube Analysis') {
            agent { label 'Slave1' } 
            steps {
                withSonarQubeEnv('SonarQube') { // Ensure you configure 'SonarQube' in Jenkins System Configuration
                    sh '''
                    mvn sonar:sonar \
                      -Dsonar.projectKey=my_project_key \
                      -Dsonar.projectName="My Project" \
                      -Dsonar.projectVersion=1.0 \
                      -Dsonar.sources=src \
                      -Dsonar.host.url=http://13.201.67.50:9000 \
                      -Dsonar.login=squ_aaea6bab6d942197fdde4bd99f0ba4789fa69198
                    '''
                }
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
        
        // Uncomment if needed
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
