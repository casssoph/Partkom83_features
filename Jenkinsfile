@Library("shared-libraries")
import io.libs.SqlUtils
import io.libs.ProjectHelpers
import io.libs.Utils

def sqlUtils = new SqlUtils()
def utils = new Utils()
def projectHelpers = new ProjectHelpers()

pipeline {
   parameters {
        string(defaultValue: "${env.jenkinsAgent}", description: 'Нода дженкинса, на которой запускать пайплайн. По умолчанию master', name: 'jenkinsAgent')
   }

    agent {
        label "${(env.jenkinsAgent == null || env.jenkinsAgent == 'null') ? "master" : env.jenkinsAgent}"
    }
    options {
        timeout(time: 8, unit: 'HOURS') 
        buildDiscarder(logRotator(numToKeepStr:'10'))
    }
    stages {
        stage("Тестирование ADD") {
            steps {
                timestamps {
                    script {
                        // Запускаем ADD тестирование на произвольной базе, сохранившейся в переменной testbaseConnString
                         returnCode = utils.cmd("runner vanessa --settings tools/vrunner.json --ordinaryapp 1")

                        if (returnCode != 0) {
                            utils.raiseError("Возникла ошибка при запуске ADD на сервере ${server1c} и базе ${testbase}")
                        }
                    }
                }
            }
        }
    }   
    post {
        always {
            script {
                if (currentBuild.result == "ABORTED") {
                    return
                }

                dir ('build/out/allure') {
                    writeFile file:'environment.properties', text:"Build=${env.BUILD_URL}"
                }

                allure includeProperties: false, jdk: '', results: [[path: 'build/out/allure']]
              
               ReportPath = "${env.WORKSPACE}/build/build/out/vbOnline.log"
         // String fileContents = new File(ReportPath).text
           String fileContents = readFile "${env.WORKSPACE}/build/build/out/vbOnline.log"
            
            }
           
   
            emailext (
        subject: "Job '${env.JOB_NAME} ${env.BUILD_NUMBER}'",
        body: fileContents,
        to: "Kalinin-VA@Part-kom.ru",
        from: "support@Part-kom.ru"
    )
           
        }
    }
}

