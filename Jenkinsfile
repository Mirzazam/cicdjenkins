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
    }

    stages{
        stage ('build'){
            steps{
                sh 'mvn -s settings.xml -DskipTests install'
            }
            
        }
    }
}