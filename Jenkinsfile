def sendEmail(job_name, job_id, test_results) {
    echo(message: "$job_name $job_id $test_results")
}

pipeline {
    agent any
    
    parameters {
      string defaultValue: 'pipeline', name: 'GIT_BRANCH', trim: true
      booleanParam defaultValue: true, name: 'RUN_TEST'
      booleanParam defaultValue: true, name: 'SEND_EMAIL'
    }

    stages {
        stage('Dynamic') {
            when {
                branch:'feature/multi*'
            }
            steps {
                echo (message: "Dynamic")
            }
        }
        stage('Download') {
            steps {
                cleanWs()
                echo (message: "Download")
                dir('pipeline') {
                    git (
                        branch: params.GIT_BRANCH,
                        url: 'https://github.com/KLevon/jenkins-course'
                    )
                }
                rtDownload (
                    serverId: 'Artifactory',
                    spec: '''{
                        "files": [
                            {
                                "pattern":"generic-local/printer.zip",
                                "target":"example-repo-local/printer.zip"
                            }
                        ]
                    }'''
                )
                unzip (
                    zipFile: "example-repo-local/printer.zip",
                    dir: "pipeline/"
                )
            }
        }
        stage('Build') {
            steps {
                echo (message: "Build")
                withCredentials (
                    [usernamePassword(credentialsId: 'iddd', passwordVariable: 'passwordd', usernameVariable: 'usernamee')]
                ) {
                    echo (message: "Credentials: $passwordd $usernamee")
                }
                bat (
                    script: """
                        cd pipeline
                        Makefile.bat
                    """
                )
            }
        }
        stage('Test') {
            when {
                equals expected: true,  
                actual: params.RUN_TEST
            }
            steps {
                echo (message: "Test")
                script {
                    def arr = ["printer", "scanner", "main"]
                    env.output = ""
                    for (el in arr) {
                        env.output += bat (
                            script: """
                                cd pipeline
                                Tests.bat $el
                            """, 
                            returnStdout: true
                        ).trim()
                    }
                }
            }
        }
        stage('Publish') {
            steps {
                echo (message: "Publish")
                script {
                    zip (
                        zipFile: "pipeline.zip",
                        archive: true,
                        dir: "pipeline"
                    )
                    rtUpload (
                        serverId: 'Artifactory',
                        spec: """{
                            "files": [
                                {
                                    "pattern":"pipeline.zip",
                                    "target":"generic-local/example-repo-local/${env.BUILD_ID}/"
                                }
                            ]
                        }"""
                    )
                }
            }
        }
    }
    post {
        always {
            script {
                if(params.SEND_EMAIL == true) {
                    sendEmail(env.JOB_NAME, env.BUILD_ID, env.output)
                }
            }
        }
    }
}
