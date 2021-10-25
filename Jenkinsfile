/* groovylint-disable-next-line CompileStatic */
pipeline {
     agent any

    stages {
        stage('Подготовка') {
            steps {
                timestamps {
                    script {
                        utils = new Utils()
                        returnCode = utils.cmd('''\"C:\\Program Files\\1cv8\\8.3.18.1208\\bin\\1cv8.exe\" CREATEINFOBASE File=\"E:\\1сработа\\kom test\\sborka\\work.database\"''')
                        if (returnCode != 0) {
                            echo "Возникла ошибка при создании базы в кластере, пытаюсь подключиться к созданной"
                        }
                        returnCode = utils.cmd('''\"C:\\Program Files\\1cv8\\8.3.18.1208\\bin\\1cv8.exe\" CONFIG /F \"E:\\1сработа\\kom test\\sborka\\work.database\" /RestoreIB "E:\\1сработа\\kom test\\piline\\dt\\1Cv8.dt"''')
                        if (returnCode != 0) {
                           echo "Возникла ошибка при загрузке dt-файла"
                        }
                        returnCode = utils.cmd('''\"C:\\Program Files\\1cv8\\8.3.18.1208\\bin\\1cv8.exe\" DESIGNER /F \"E:\\1сработа\\kom test\\sborka\\work.database\" /ConfigurationRepositoryF \"E:\\1сработа\\kom test\\хранилище\" /ConfigurationRepositoryN \"test\" /ConfigurationRepositoryP \"test\" /ConfigurationRepositoryUpdateCfg -force /UpdateDBCfg  -Dunamic /Out report /DisableStartupMessages /DisableStartupDialogs''')
                        if (returnCode != 0) {
                            utils.raiseError("Возникла ошибка при подключении к хранилищу")
                        }
                       returnCode = utils.cmd('''\"C:\\Program Files\\1cv8\\8.3.18.1208\\bin\\1cv8.exe\" ENTERPRISE /F \"E:\\1сработа\\kom test\\sborka\\work.database\"''')
                       if (returnCode != 0) {
                            utils.raiseError("Не удалось запустить первый раз")
                       }
                       returnCode = utils.cmd('''ping -n 10 localhost > Null''')
                       if (returnCode != 0) {
                            utils.raiseError("Пауза не удалась")
                       }
                       returnCode = utils.cmd('''\"C:\\Program Files\\1cv8\\8.3.18.1208\\bin\\1cv8.exe\" ENTERPRISE /F \"E:\\1сработа\\kom test\\sborka\\work.database\"  /N "Администратор" /P "" /Execute \"E:\\1сработа\\kom test\\piline\\fixtures\\fixtures.epf\"''')
                       if (returnCode != 0) {
                            utils.raiseError("Возникла ошибка настройки базы")
                       }
                       returnCode = utils.cmd('''\"C:\\Program Files\\1cv8\\8.3.18.1208\\bin\\1cv8.exe\" ENTERPRISE /F \"E:\\1сработа\\kom test\\sborka\\work.database\"  /N "Администратор" /P "" /Execute \"E:\\1сработа\\Git\\xUnit\\xddTestRunner.epf\" /C \"xddRun ЗагрузчикКаталога ""E:\\tests\\"";  xddReport ГенераторОтчетаAllureXML ""E:\\1сработа\\kom test\\reports\\report-allure.xml""; xddShutdown;\"''')
                       if (returnCode != 0) {
                            utils.raiseError("Возникла ошибка запуска обработки xUnit")
                       }
                    }
                }
            }
        }
        stage('test') {
            steps {
                echo 'nu test'
            }
        }
    }
}
