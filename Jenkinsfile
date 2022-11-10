import groovy.json.JsonSlurper

pipeline {
    agent any

    stages {
        stage("PreSetup") {
            steps{
                script{
                    def response = httpRequest url: "https://owl-flex-store-7157-dev.twil.io/generate-token?username=${HTTP_USERNAME}&password=${HTTP_PASSWORD}", wrapAsMultipart: false
                    println('Status: '+response.status)
                    if(response.status == 200) {
                        def jsonSlurper = new JsonSlurper() 
                        def object = jsonSlurper.parseText(response.content)
                        println('Access Token: '+ object.access_token)
                        env.TWIL_IO_ACCESS_TOKEN = object.access_token
                    }
                }
            }
        }
        stage("Setup") {
            steps {
                echo "Configuring Envronment"
                dir("plugin-auto-answer-call") {
                    script {
                        env.REACT_APP_ANNOUNCE_MEDIA = "${REACT_APP_ANNOUNCE_MEDIA}"
                    }
                }
            }
        }
        stage("Flex") {
            stages {
                stage("Notification") {
                    steps{
                        script {
                            def response = httpRequest customHeaders: [[name: 'Authorization', value: "Bearer ${TWIL_IO_ACCESS_TOKEN}"]], url: "https://owl-flex-store-7157-dev.twil.io/send-email-notification?emailAddress=${EMAIL}&accountSid=${TWILIO_ACCOUNT_SID}&pluginName=plugin-${JOB_NAME}&statusMessage=Building&pluginHeader=${JOB_NAME}", wrapAsMultipart: false
                            println('Status: '+response.status)
                        }
                    }
                }
                stage("Build") {
                    steps {
                        echo "Building..."
                        dir("plugin-auto-answer-call") {
                            echo pwd()
                            nodejs("Node-16.18.0") {
                                sh "twilio plugins:install @twilio-labs/plugin-flex"
                                sh "npm install"
                                sh "twilio flex:plugins:build"
                            }
                        }
                    }
                }
                stage("Test") {
                    steps {
                        echo "Testing..."
                        dir("plugin-auto-answer-call") {
                            echo pwd()
                        }
                    }
                }
                stage("Deploy") {
                    steps {
                        echo "Deploying..."
                        dir("plugin-auto-answer-call") {
                            echo pwd()
                            nodejs("Node-16.18.0") {
                                sh "twilio flex:plugins:deploy --major --changelog 'One-Click-Deploy' --description 'Sample OOTB Twilio Flex Plugin'"
                                script {
                                    if(fileExists("package.json")) {
                                        packageJSON = readJSON(file:"package.json");
                                    }
                                 }
                                sh "twilio flex:plugins:release --plugin ${packageJSON.name}@${packageJSON.version} --name 'plugin-auto-answer-call' --description 'Releasing auto answer call plugin'"
                            }
                        }
                    }
                }
                stage("Post Deploy") {
                    steps {
                        echo "Post Deployment"
                        script {
                            def response = httpRequest customHeaders: [[name: 'Authorization', value: "Bearer ${TWIL_IO_ACCESS_TOKEN}"]], url: "https://owl-flex-store-7157-dev.twil.io/send-email-notification?emailAddress=${EMAIL}&accountSid=${TWILIO_ACCOUNT_SID}&pluginName=plugin-${JOB_NAME}&statusMessage=Deployed&pluginHeader=${JOB_NAME}", wrapAsMultipart: false
                            println('Status: '+response.status)
                        }
                    }
                }
            }
        }
    }
}
