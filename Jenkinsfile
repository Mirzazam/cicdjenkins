pipeline{
    agent any
    tools{
        maven "MAVEN3"
        jdk "OracleJDK8"
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
    }
}