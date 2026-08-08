pipeline {
    agent any
    stages {
        stage('checkout') {
            steps {
                git 'https://github.com/SubhamBidhan/java-project-maven-new.git'
            }
        }
        stage('compile') {
            steps {
                sh 'mvn compile'
            }
        }
        stage('test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('package') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('SonarQube') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.7.0.1746:sonar'
                }
            }
        }
        stage('Nexus-Artifact') {
            steps {
                nexusArtifactUploader artifacts: [[
                    artifactId: 'myapp',
                    classifier: '',
                    file: 'target/myapp.war',
                    type: '.war'
                ]],
                credentialsId: 'nexus',
                groupId: 'in.reyaz',
                nexusUrl: '<NEXUS_IP>:8081',
                nexusVersion: 'nexus3',
                protocol: 'http',
                repository: 'hotstarapp',
                version: '8.3.3-SNAPSHOT'
            }
        }
        stage('deploy') {
            steps {
                ansiblePlaybook(
                    playbook: '/etc/ansible/deploy.yml',
                    inventory: '/etc/ansible/hosts',
                    credentialsId: 'Linux_creds',
                    disableHostKeyChecking: true
                )
            }
        }
    }
}
