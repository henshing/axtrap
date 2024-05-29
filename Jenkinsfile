pipeline {
    agent any

    parameters{
        string(name:'mainRepo',defaultValue:'https://github.com/henshing/Starry.git',description:'main repository')
        string(name:'relatedRepo1',defaultValue:'https://github.com/henshing/driver_display.git',description:'related repository')
        string(name:'relatedRepo2',defaultValue:'https://github.com/henshing/axtrap.git',description:'related repository')
    }

    environment {
        JENKINS_URL = "http://49.51.192.19:9095"
        JOB_PATH = "job/github_test_sl"
        REPORT_PATH = "allure"
    }
    
    stages {
        stage('MainRepoTest'){
            steps{
                sh"git clone ${parameters.mainRepo};cd${parameters.mainRepo};pytest"
            }
        }
        
        stage('RelatedRepoTest'){
            steps{
                sh"git clone ${parameters.relatedRepo1};cd${parameters.relatedRepo1};pytest"
            }
        }

        stage('RelatedRepoTest'){
            steps{
                sh"git clone ${parameters.relatedRepo2};cd${parameters.relatedRepo2};pytest"
            }
        }
        
        stage('pytest嵌入'){
                    steps{
                            sh 'echo $PATH'
                            sh 'printenv'
                            sh 'cp -r /home/jenkins_home/pytest $WORKSPACE'
                    }
                }
        
        stage('Report') {
            steps {
                script {
                    //根据内置变量currentBuild获取构建号
                    def buildNumber = currentBuild.number
                    // 构建 Allure 报告地址
                    def allureReportUrl = "${JENKINS_URL}/${JOB_PATH}/${buildNumber}/${REPORT_PATH}"
                    
                    // 输出 Allure 报告地址
                    echo "Allure Report URL: ${allureReportUrl}"
                }
            }
        }

        stage('结果展示'){
                    steps{
                        echo "-------------------------allure report generating start---------------------------------------------------"
                        sh 'export pywork=$WORKSPACE && cd $pywork/pytest && allure generate ./report/result -o ./report/html --clean'
                        allure includeProperties: false, jdk: 'jdk21', report: 'pytest/report/html', results: [[path: 'pytest/report/result']]
                        echo "-------------------------allure report generating end ----------------------------------------------------"
                    }
                }
    }

    post {
        always {
            junit'**/target/*.xml'
            script {
                
                allure includeProperties: false, jdk: '', reportBuildPolicy: 'ALWAYS', results: [[path: 'target/allure-results']]
            }
        }
        failure{
            mail to:team@example.com,subject:'The pipeline failed :('
        }
        success{
            script{
                mail to:"${parameters.email}",
                subject:"PipeLine'${JOB_NAME}'(${BUILD_NUMBER})result",
                body:"${env.name}\n pipeline '${JOB_NAME}'(${BUILD_NUMBER}) (${allureReportUrl})"
            }
        }
    }
}
