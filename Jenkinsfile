pipeline {
    agent any
    environment {
        SONAR_URL ='http://10.182.0.12:9000'
        SONAR_TOKEN = credentials('sonar-jenkins')
        SONAR_PROJECT_KEY = 'sonar-jenkins'
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "10.182.0.11:8081"
        NEXUS_REPOSITORY = "maven-demo"
        NEXUS_CREDENTIAL_ID = "jenkins-sonar-nexsus"
    }

    stages{
        stage('git checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Sureshreddybhemi/maven-demo.git'
            }
        }
        stage('inject enivornment variable') {
            steps {
                script {
                    env.NEXUSSERVERIP = '10.182.0.11'
                }
            }
        }
        stage('sonarqube scan') {
            steps {
                script {
                    withSonarQubeEnv('SonarQubeServer') {
                        // Run SonarQube scanner with parameters
                        sh "mvn sonar:sonar \
                            -Dsonar.host.url=${SONAR_URL} \
                            -Dsonar.login=${SONAR_TOKEN} \
                            -Dsonar.projectKey=${SONAR_PROJECT_KEY}"
                    }
                }
            }
        }
        stage('maven build') {
            steps {
                script {
                    env.NEXUSSERVERIP = "${NEXUSSERVERIP}"
                }
                sh 'mvn clean package'
            }
        }
        stage('nexus publish') {
            steps {
                script {
                    pom = readMavenPom file: "pom.xml"
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}")
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    artifactPath = filesByGlob[0].path
                    artifactExists = fileExists artifactPath
                    if (artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}"
                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                [artifactId: pom.artifactId,
                                    classifier: '',
                                    file: artifactPath,
                                    type: pom.packaging],
                                [artifactId: pom.artifactId,
                                    classifier: '',
                                    file: "pom.xml",
                                    type: "pom"]
                            ]
                        )
                    } else {
                        error "*** File: ${artifactPath}, could not be found"
                    }
                }
            }
        }
    }
}
