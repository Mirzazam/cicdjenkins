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
        SNAP_REPO = 'vprofile-snapshots'
        NEXUS_USER = "admin"
        NEXUS_PASS = "admin"
        RELEASE_REPO = "vprofile-release"
        CENTRAL_REPO = "vpro-maven-central"
        NEXUS_GRP_REPO = "vpro-maven-group"
        NEXUSIP = "172.31.18.208"
        NEXUSPORT = "8081"
        NEXUS_LOGIN = "nexuslogin"
        SONARSERVER = 'sonarserver'
        SONARSCANNER = 'sonarscanner'
        NEXUSPASS= credentials('nexuspass')
    }

    stages{
        stage('clean, compile, test,package, install, checkstyle'){
            steps{
                sh 'mvn clean'
                sh 'mvn compile'
                sh ' mvn test-compile'
                sh 'mvn test'
                sh 'mvn package'
                sh 'mvn -U clean install'
                sh 'mvn -s settings.xml -DskipTest install'
                sh 'mvn checkstyle:checkstyle'
            }
        }

        stage('CODE ANALYSIS with SONARQUBE') {
          
		  environment {
             scannerHome = tool "${SONARSCANNER}"
          }

          steps {
            withSonarQubeEnv('sonarserver') {
               sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
            }
         }  
         post {
            success {
                echo 'Now Archiving...'
                archiveArtifacts artifacts: '**/target/*.war'
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

    stage('Uploading Artifact to Nexus Repo') {
        steps{
        nexusArtifactUploader(
        nexusVersion: 'nexus3',
        protocol: 'http',
        nexusUrl: "${NEXUSIP}:${NEXUSPORT}",
        groupId: 'quality check',
        version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
        repository: "${RELEASE_REPO}",
        credentialsId: "${NEXUS_LOGIN}",
        artifacts: [
            [artifactId: 'vproapp',
             classifier: '',
             file: 'target/vprofile-v2.war',
             type: 'war']
        ]
     )
        }
    }

    stage("Deploy anisble playbook"){
        steps{
            ansiblePlaybook([
                inventory: 'ansible/stage.inventory',
                playbook: 'ansible/site.yml',
                installation: 'ansible',
                colorized: true,
                credentialsId: 'applogin',
                disableHostKeyChecking: true,
                extraVars: [
                    USER: "admin",
                    PASS: ${NEXUSPASS},
                    nexusip: '172.31.18.208',
                    reponame: "vprofile-release",
                    groupid: 'quality check',
                    time: "${env.BUILD_TIMESTAMP}",
                    build: "${env.BUILD_ID}",
                    artifactid: "vproapp",
                    vprofile_version: "vproapp-${env.BUILD_ID}-${env.BUILD.TIMESTAMP}.war"

                ]
            ])
        }
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

