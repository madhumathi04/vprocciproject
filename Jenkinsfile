pipeline{
    agent any
    tools {
        maven "MAVEN3"
        jdk "OracleJDK8"
    }

    environment {
         SNAP_REPO = 'vprofile-snapshot'
         NEXUS_USER = 'admin'
         NEXUS_PASS = 'admin123'
         RELEASE_REPO = 'vprofile-release'
         CENTRAL_REPO = 'vpro-maven-central'
         NEXUSIP = '3.87.231.175'
         NEXUSPORT = '8081'
         NEXUS_GRP_REPO = 'vpro-maven-group'
         NEXUS_LOGIN = 'nexuslogin' 
         SONARSERVER = 'sonarserver'
         SONARSCANNER = 'sonarscanner' 
        registryCredential = 'ecr:us-west-1:awscreds'
        appRegistry = '951401132355.dkr.ecr.us-west-1.amazonaws.com/vprofileappimg'
        vprofileRegistry = "https://951401132355.dkr.ecr.us-west-1.amazonaws.com"
        cluster = "vprostaging"
        service = "vproappprodsvc" 
        }
    stages {
        stage('Build') {
            steps {
               sh 'mvn clean install -U -DskipTests -Dmaven.repo.local=~/.m2/repository'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }
            stage('UNIT TEST'){
            steps {
                sh 'mvn clean install -U -DskipTests -Dmaven.repo.local=~/.m2/repository test'
            }
        }    
            stage ('Checkstyle Analysis'){
            steps {
                sh 'mvn clean install -U -DskipTests -Dmaven.repo.local=~/.m2/repository checkstyle:checkstyle'
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
        stage ('Quality Gate') {
        steps {
            timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: true
            }
        }
    }
    stage("UploadArtifact"){
            steps{
                nexusArtifactUploader(
                  nexusVersion: 'nexus3',
                  protocol: 'http',
                  nexusUrl: "${NEXUSIP}:${NEXUSPORT}",
                  groupId: 'QA',
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
        stage('Ansible Deploy to Prod'){
            Steps {
	            ansiblePlaybook([
	            inventory : 'ansible/prod.inventory',
	            playbook  : 'ansible/site.yml'
                installation : 'ansible',
                colorized : true,
                credentialsId: 	'applogin',
                disableHostKeyChecking: true,
                extraVars   :  [
                    USER: "admin",
                    PASS: "admin123",
                    nexusip: "54.158.142.54"
                    reponame: "vprofile-release",
                    groupid: "QA"
                    time: "$(env.TIME)"
                    build: "$(env.BUILD)",
                    artifactid: "vproapp",
                    vprofile_version: "vproapp-${env.BUILD}-${env.BUILD_TIME}.war"
                ]
            ])
	    }
    }
    post {
        always {
            echo 'Slack Notifications.'
            slackSend channel: '#jenkinscicd',
                color: COLOR_MAP[currentBuild.currentResult],
                message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
        }
    }
}

    
