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
                scannerHome = tool "${SONARSCANNER}"
            }
            steps {
                withSonarQubeEnv('sonarserver') { 
                        sh '''${sonarsHome}/bin/sonar-scanner -Dsonar.projectkey=vprofile \
                        -Dsonar.projectname=vprofile \
                        -Dsonar.sources=src/ \
                        -Dsonar.java.checkstyle.reportsPath=target/checkstyle-result.xml \
                        -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                        -Dsonar.junit.reportsPath=target/surefire-reports/ '''
                
                }
                        
                
            }
        }
    }
        
}
    

