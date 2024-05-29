pipeline {
    agent any

    parameters{
        string(name:'mainRepo',defaultValue:'https://github.com/henshing/Starry.git',description:'main repository')
        string(name:'relatedRepo1',defaultValue:'https://github.com/henshing/driver_display.git',description:'related repository')
        string(name:'relatedRepo2',defaultValue:'https://github.com/henshing/axtrap.git',description:'related repository')
        string(name:'email',defaultValue:'1445323887@qq.com',description:'Email address to send the report to')
    }

    environment {
        JENKINS_URL = "http://49.51.192.19:9095"
        JOB_PATH = "job/github_test_sl"
        REPORT_PATH = "allure"
    }
    
    stages {
        stage('Setup Environment') {
            steps {
                script {
                    // 获取当前构建号
                    def buildNumber = currentBuild.number
                    // 构建 Allure 报告地址
                    env.ALLURE_REPORT_URL = "${env.JENKINS_URL}/${env.JOB_PATH}/${buildNumber}/${env.REPORT_PATH}"
                    // 输出 Allure 报告地址
                    echo "Allure Report URL: ${env.ALLURE_REPORT_URL}"
                }
            }
        }

        stage('MainRepoTest'){
            steps{
                sh "git clone ${params.mainRepo}; cd ${params.mainRepo}; pytest"
            }
        }
        
        stage('RelatedRepoTest1'){
            steps{
                sh "git clone ${params.relatedRepo1}; cd ${params.relatedRepo1}; pytest"
            }
        }

        stage('RelatedRepoTest2'){
            steps{
                sh "git clone ${params.relatedRepo2}; cd ${params.relatedRepo2}; pytest"
            }
        }
        
        stage('pytest嵌入'){
            steps{
                sh 'echo $PATH'
                sh 'printenv'
                sh 'cp -r /home/jenkins_home/pytest $WORKSPACE'
            }
        }

        stage('编译测试'){
            steps {
                echo "--------------------------------------------test start------------------------------------------------"
                sh ' export pywork=$WORKSPACE && cd $pywork/pytest && python3 -m pytest -sv --alluredir report/result testcase/test_arceos.py --clean-alluredir'
                echo "--------------------------------------------test end  ------------------------------------------------"
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
            junit '**/target/*.xml'
        }
        failure {
            emailext(
                to: "${params.email}",
                subject: "The pipeline failed",
                body: """
                你好，

                流水线 '${JOB_NAME}' 的构建失败了。构建号：${BUILD_NUMBER}。

                请查看以下日志以了解详情。

                谢谢，
                Jenkins
                """
            )
        }
        success {
            emailext(
                to: "${params.email}",
                subject: "The pipeline succeeded",
                body: """
                你好，

                流水线 '${JOB_NAME}' 的构建已成功完成。构建号：${BUILD_NUMBER}。

                请查看以下报告：
                ${env.ALLURE_REPORT_URL}

                环境名称：${env.name}

                谢谢，
                Jenkins
                """
            )
        }
    }
}
