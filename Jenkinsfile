pipeline {
    agent any
    
    environment {
        APP_NAME = 'ExtenSURE Tooling'
        BUILD_NUMBER = "${env.BUILD_NUMBER}"
        issues = ""
        analyses = ""
        metrics = ""
    }

    parameters {
        string(defaultValue: "git@gitext.persistent.co.in:SCMGroup/extensure_tooling.git", description: 'Path to Git repository', name: 'GIT_REPO_URL')
        string(defaultValue: "develop", description: 'Branch Specifier', name: 'BRANCH_SPECIFIER')
        string(defaultValue: "extensure-jenkins-ssh", description: 'Credentails ID to access SCM (if required)', name: 'GIT_CRED_ID')
        string(defaultValue: "ExtenSURE_Tooling", description: 'Project Name', name: 'PROJECT_NAME')
    }

    stages {
        stage('Initialize') {
            steps {
                script {
                    echo "${BUILD_NUMBER} - ${env.BUILD_ID} on ${env.JENKINS_URL}"
                    echo "GIT Repo URL :: ${params.GIT_REPO_URL}"
                    echo "GIT Branch   :: ${params.BRANCH_SPECIFIER}"
                }
            }
        }

        stage("Clone Git Repo") {
            steps {
                git branch: "${params.BRANCH_SPECIFIER}",
                credentialsId: "${GIT_CRED_ID}",
                url: "${GIT_REPO_URL}"
                echo "***** Code Checkout from Git SCM completed *****"
            }
        }

        stage('Execute SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'extensure-sonar-scanner';
                    withSonarQubeEnv(installationName: 'extensure-sonar-container', credentialsID: 'extensure-sonar-token')
                    {
                        sh "${tool("extensure-sonar-scanner")}/bin/sonar-scanner \
                        -Dsonar.projectKey='${PROJECT_NAME}' \
                        -Dsonar.sources=./src "
                    }
                }
                echo "***** SonarQube Analysis completed *****"
            }
        }

        stage("Quality Gate") {
            steps {
                sleep(10) // workaround else wait will time out
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
                echo "***** Quality Gate check completed *****"
            }
        }

        stage("Collate Metrics") {
            steps {
                script {
                    issues = new URL("http://localhost:9000/api/issues/search?componentKeys=${PROJECT_NAME}").openConnection().getInputStream().getText();
                    analyses = new URL("http://localhost:9000/api/project_analyses/search?project=${PROJECT_NAME}").openConnection().getInputStream().getText();
                    metrics = new URL("http://localhost:9000/api/metrics/search").openConnection().getInputStream().getText();
                }
                echo "***** Collation of Metrics completed *****"
            }
        }
    
        stage("Publish Reports") {
            steps {
                script {
                    issues_file = new File("$WORKSPACE/"+"${PROJECT_NAME}_Issues.json")
                    analyses_file = new File("$WORKSPACE/"+"${PROJECT_NAME}_Analyses.json")
                    metrics_file = new File("$WORKSPACE/"+"${PROJECT_NAME}_Metrics.json")
                    issues_file.write(issues)
                    analyses_file.write(analyses)
                    metrics_file.write(metrics)
                }
                echo "***** Publish Reports completed *****"
            }
        }    
    }
}