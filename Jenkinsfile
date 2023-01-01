def COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger'
]

pipeline{
    agent any

    tools{
        maven "MAVEN3"
        jdk "oraclejdk8"
    }

    environment{
        SNAP_REPO= "vpro-snap"
        NEXUS_USER= "admin"
        NEXUS_PASS= "admin"
        RELEASE_REPO= "vpro-release"
        CENTRAL_REPO= "vpro-dependency"
        NEXUS_GRP_REPO= "vpro-group-repo"
        NEXUSIP= "172.31.16.35"
        NEXUSPORT= "8081"
        NEXUS_LOGIN= "nexuslogin"
        SONARTOOL= "sonartool"
    }

    stages{
        stage("Clean,Compile,Compile-test,Test,Install"){
            steps{
                sh 'mvn clean'
            sh 'mvn compile'
            sh 'mvn compile-test'
            sh 'mvn test'
            sh 'mvn package'
            sh 'mvn -s settings.xml -DskipTest install'
            sh 'mvn checkstyle:checkstyle'
            }
        }

        stage("Code analysis with Sonarscanner"){
            environment{
                scannerHome = tool "${SONARTOOL}"
            }
                steps{
                    sh ''' "${SONARTOOL}/bin/sonar-scanner -Dsonar.projectkey= vprofile/ \
                    -Dsonar.projectName=vprofile-repo \
                    -Dsonar.projectVersion=1.0 \
                    -Dsonar.sources=src/ \
                    -Dsonar.Java.binaries=target/test-classes/com/visualpathit/accounts/ControllerTest/ \
                    -Dsonar.junit.reportsPath=target/surefire-reports/ \
                    -Dsonar.jacoco.reportsPath=target/jacoco.exec/ \
                    -Dsonar.java.checkstyle.reportsPaths=target/checkstyle-result.xml'''                
                }

        }

        stage("Quality gate") {
            steps {
                waitForQualityGate abortPipeline: true
            }
        }

        stage("Upload Artifact to Nexus repo"){
            steps{
            nexusArtifactUploader(
            nexusVersion: 'nexus3',
            protocol: 'http',
            nexusUrl: "${NEXUSIP}:${NEXUSPORT}",
            groupId: 'checking repo',
            version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
            repository: 'RepositoryName',
            credentialsId: 'nexuslogin',
            artifacts: [
                [artifactId: vpro,
                classifier: '',
                file: 'target/vprofile-v2.war',
                type: 'war']
            ]
        )
            }
        }

        
        post {
        always {
            echo 'Slack Notifications.'
            slackSend channel: '#dev',
            color: COLOR_MAP[currentBuild.currentResult],
            message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
        }
    }  
    }

    
}





