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
    }

    stage("build & SonarQube analysis") {
          node {
              withSonarQubeEnv('sonarserver') {
                 sh 'mvn clean package sonar:sonar'
              }
          }
      }

      stage("Quality Gate"){
          timeout(time: 1, unit: 'HOURS') {
              def qg = waitForQualityGate()
              if (qg.status != 'OK') {
                  error "Pipeline aborted due to quality gate failure: ${qg.status}"
              }
          }
      }

}