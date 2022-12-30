pipeline{
    agent any
    tools{
        maven "MAVEN3"
        jdk "oraclejdk8"
    }

    environment{
        SNAP_REPO = 'vprofile-snapshots'
        NEXUS_USER = "admin"
        NEXUS_PASS = "admin"
        RELEASE_REPO = "vprofile-release"
        CENTRAL_REPO = "vpro-maven-central"
        NEXUS_GRP_REPO = "vpro-maven-group"
        NEXUSIP = "172.31.22.12"
        NEXUSPORT = "8081"
        NEXUS_LOGIN = "nexuslogin"
        SONARSERVER = 'sonarserver'
        SONARSCANNER = 'sonarscanner'
    }

    stages{
        stage('all commands'){
            steps{
                sh 'mvn clean'
                sh 'mvn compile'
                sh ' mvn test-compile'
                sh 'mvn test'
                sh 'mvn package'
                sh 'mvn -s settings.xml -DskipTest install'
                sh 'mvn checkstyle:checkstyle'
            }
        }

        stage('build && SonarQube analysis') {
            environment {
                scannerHome = tool 'sonarscanner'
            }
            steps {
                withSonarQubeEnv('sonarserver') {
                    // Optionally use a Maven environment you've configured already
                    withMaven(maven:'MAVEN3') {
                        sh 'mvn clean package sonar:sonar'
                    }
                }
            }
        }
        stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
    

}