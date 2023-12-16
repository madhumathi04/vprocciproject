def COLOR_MAP = [
    'SUCCESS' : 'good',
    'FAILURE' : 'danger',
]
pipeline{
   agent any
   environment{
      NEXSUSPASS = credentials('nexuspass')
    }

    stages{

        stage('Setup parameters'){
            steps {
                script {
                    properties([
                        parameters([
                            string(
                            defualtValue:'',
                             name: 'BUILD',
                            ),
                            string(
                               defualtValue:'',
                                name: 'TIME',
                            )
                        ])
                    ])
                }
            }
        }
        stage('Ansible Deploy to Prod') {
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
                    groupid: "QA",
                    time: "\$(env.TIME)",
                    build: "\$(env.BUILD)",
                    artifactid: "vproapp",
                    vprofile_version: "vproapp-${env.BUILD}-${env.TIME}.war"
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

