pipeline {
    agent {
        label 'AGENT-1' 
    }
    options {
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
        ansiColor('xterm')
    }
    environment {
        def appVersion = '' //declare global variable here so that this can be used across all stages
        def nexusUrl = 'nexus.avyan.site:8081' // this has nexus private id address.. jenkins-agent instance connects to Nexus instance using private ip 
    }
    stages {
        stage('read the version') {
            steps {
                script {
                    def packageJson = readJSON file : 'package.json'
                    appVersion = packageJson.version
                    echo "application version --: $appVersion"
                }
            }
        }
        stage('Build') {
            steps {
                sh """
                    zip -r -q frontend-${appVersion}.zip * -x Jenkinsfile -x frontend-${appVersion}.zip
                    ls -ltr
                """
            }
        }
        stage('Nexus Artifact Upload'){
            steps{
                script{
                    nexusArtifactUploader(
                        nexusVersion: 'nexus3',
                        protocol: 'http',
                        nexusUrl: "${nexusUrl}",
                        groupId: 'com.expense', // create this group id folder
                        version: "${appVersion}", // create folder with this version inside artifact id folder and store artifact there
                        repository: "frontend", // store artifact in this repo created manually in Nexus console
                        credentialsId: 'nexus-auth', // created credentials in Jenkins console to access Nexus from Jenkins-agent
                        artifacts: [
                            [artifactId: "frontend" , // create this artifact id folder inside group id folder
                            classifier: '',
                            file: "frontend-" + "${appVersion}" + '.zip', // artifact file name to be stored
                            type: 'zip']
                        ]
                    )
                }
            }
        }
        stage('Deploy'){
            steps{
                script{
                    def params = [
                        string(name: 'appVersion', value: "${appVersion}") //these params will be used in frontend-deploy-using-jenkins project
                    ]
                    build job: 'frontend-deploy', parameters: params, wait: false // this will call frontend-deploy pipeline created in jenkins for deployment which has frontend-deploy-using-jenkins projec git url
                    // you can keep wait = true here if you want CI job to fail if deploy failed 
                }
            }
        }
    }

    post {
        always {
            echo "I will always run even when pipeline is success or failure"
            deleteDir()
        }
        success {
            echo "I will run when pipeline is success"
        }
        failure {
            echo "I will run when pipeline is failure"
        }
    }
}