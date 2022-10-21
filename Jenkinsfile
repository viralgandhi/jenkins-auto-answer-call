pipeline {
    agent any

    stages {
        stage("Setup") {
            steps {
                echo "Configuring Envronment"
                dir("plugin-auto-answer-call") {
                    script {
                        env.REACT_APP_ANNOUNCE_MEDIA = "https://api.twilio.com/cowbell.mp3"
                    }
                }
            }
        }
        stage("Flex") {
            stages {
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
                    }
                }
            }
        }
    }
}