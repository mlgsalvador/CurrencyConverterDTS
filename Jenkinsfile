pipeline {
    agent any
    stages {
        stage('Build') {
            environment {
                mvnHome = tool name: 'ADOP Maven', type: 'maven'
                jdk = tool name: 'Java', type: 'jdk'
            steps {
                cleanWs()

                checkout changelog: false,
                    poll: false,
                    scm: [$class: 'GitSCM',
                    branches: [[name: '*/master']],
                    doGenerateSubmoduleConfigurations: false,
                    extensions: [],
                    submoduleCfg: [],
                    url: 'https://github.com/mlgsalvador/CurrencyConverterDTS.git']]]       
                 

                mavenJob('CurrencyConverter') {
                    scm {
                        github('mlgsalvador/CurrencyConverterDTS', 'master', 'https', 'github.com')
                    }
                    triggers {
                        githubPush()
                        buildOnMergeRequestEvents(true)
                        buildOnPushEvents(true)
                        enableCiSkip(false)
                        setBuildDescription(false)
                        rebuildOpenMergeRequest('never')
                        includeBranches('master')
                    }
                    goals('package')
                    archiveArtifacts '**/*.war'
                }
                }
            }
        }
    }
}    