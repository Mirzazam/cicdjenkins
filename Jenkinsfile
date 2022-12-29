pipeline{
    agent any
    tools{
        maven "maven"
        jdk "oracle8"
    }

    environment{
        SNAP_REPO= 'vprofile-snapshot'
        NEXUS_USER= 'admin'
        NEXUS_PASS= 'admin'
        RELEASE_REPO= 'vprofile-release'
        CENTRAL_REPO= 'vpro-maven-central'
        NEXUSPORT= '8081'
        NEXUSIP= '172.31.1.9'
        NEXUS_GRP_REPO= 'vpro-maven-group'
        NEXUS_LOGIN= 'nexusulogin'
        SONARSERVER= 'sonarserver'
        SONARSCANNER= 'sonarscanner'
    }

    stages{
        stage('build'){
            steps {
                sh 'mvn -s settings.xml -Dskiptests install'
            }
            post{
                success{
                    echo "now archiving"
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }
        stage('test'){
            steps{
                sh 'mvn -s settings.xml test'
            }
        }
        stage('checkstyle analysis'){
            steps{
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
        }

        stage('CODE ANALYSIS with SONARQUBE') {
          
		  environment {
             scannerHome = tool "${SONARSCANNER}"
           }

          steps {
            withSonarQubeEnv("${SONARSERVER}") {
               sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile-repo \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }
            }
        }
    }
}