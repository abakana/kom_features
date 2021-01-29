@Library("shared-libraries")
import io.libs.SqlUtils
import io.libs.ProjectHelpers
import io.libs.Utils

def sqlUtils = new SqlUtils()
def utils = new Utils()
def projectHelpers = new ProjectHelpers()
def backupTasks = [:]
def restoreTasks = [:]
def dropDbTasks = [:]
def createDbTasks = [:]
def runHandlers1cTasks = [:]
def updateDbTasks = [:]

pipeline
{

    agent {
        label "${(env.jenkinsAgent == null || env.jenkinsAgent == 'null') ? "master" : env.jenkinsAgent}"
    }
    options {
        timeout(time: 8, unit: 'HOURS')
        buildDiscarder(logRotator(numToKeepStr:'10'))
    }
    stages {
        stage("Подготовка") {
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
                        //returnCode = utils.cmd('''\"C:\\Program Files\\1cv8\\8.3.18.1208\\bin\\1cv8.exe\" DESIGNER /F \"E:\\1сработа\\kom test\\sborka\\work.database\"  /ConfigurationRepositoryUpdateCfg -force /UpdateDBCfg  -Dunamic /Out report /DisableStartupMessages /DisableStartupDialogs /LoadCfg \"E:\\1сработа\\kom test\\piline\\cfe\\ОтключениеПроверкиЛицензии.cfe" -Extension \"ОтключениеЛицензирования\"''')
                        //if (returnCode != 0) {
                        //    utils.raiseError("Возникла ошибка при подключении расширения")
                        //}
                       //returnCode = utils.cmd('''\"C:\\Program Files\\1cv8\\8.3.18.1208\\bin\\1cv8.exe\" ENTERPRISE /F \"E:\\1сработа\\kom test\\sborka\\work.database\" /Execute \"E:\\1сработа\\kom test\\piline\\fixtures\\fixtures.epf\"''')
                        //if (returnCode != 0) {
                        //    utils.raiseError("Возникла ошибка настройке расширения")
                        //}
                    }
                }
            }
        }
    }
}

def dropDbTask(server1c, server1cPort, serverSql, infobase, admin1cUser, admin1cPwd, sqluser, sqlPwd) {
    return {
        timestamps {
            stage("Удаление ${infobase}") {
                def projectHelpers = new ProjectHelpers()
                def utils = new Utils()

                projectHelpers.dropDb(server1c, server1cPort, serverSql, infobase, admin1cUser, admin1cPwd, sqluser, sqlPwd)
            }
        }
    }
}

def createDbTask(server1c, serverSql, platform1c, infobase) {
    return {
        stage("Создание базы ${infobase}") {
            timestamps {
                def projectHelpers = new ProjectHelpers()
                try {
                    projectHelpers.createDb(platform1c, server1c, serversql, infobase, null, false)
                } catch (excp) {
                    echo "Error happened when creating base ${infobase}. Probably base already exists in the ibases.v8i list. Skip the error"
                }
            }
        }
    }
}

def backupTask(serverSql, infobase, backupPath, sqlUser, sqlPwd) {
    return {
        stage("sql бекап ${infobase}") {
            timestamps {
                def sqlUtils = new SqlUtils()

                sqlUtils.checkDb(serverSql, infobase, sqlUser, sqlPwd)
                sqlUtils.backupDb(serverSql, infobase, backupPath, sqlUser, sqlPwd)
            }
        }
    }
}

def restoreTask(serverSql, infobase, backupPath, sqlUser, sqlPwd) {
    return {
        stage("Востановление ${infobase} бекапа") {
            timestamps {
                sqlUtils = new SqlUtils()

                sqlUtils.createEmptyDb(serverSql, infobase, sqlUser, sqlPwd)
                sqlUtils.restoreDb(serverSql, infobase, backupPath, sqlUser, sqlPwd)
            }
        }
    }
}

def runHandlers1cTask(infobase, admin1cUser, admin1cPwd, testbaseConnString) {
    return {
        stage("Запуск 1с обработки на ${infobase}") {
            timestamps {
                def projectHelpers = new ProjectHelpers()
                projectHelpers.unlocking1cBase(testbaseConnString, admin1cUser, admin1cPwd)
            }
        }
    }
}

def updateDbTask(platform1c, infobase, storage1cPath, storageUser, storagePwd, connString, admin1cUser, admin1cPwd) {
    return {
        stage("Загрузка из хранилища ${infobase}") {
            timestamps {
                prHelpers = new ProjectHelpers()

                if (storage1cPath == null || storage1cPath.isEmpty()) {
                    return
                }

                prHelpers.loadCfgFrom1CStorage(storage1cPath, storageUser, storagePwd, connString, admin1cUser, admin1cPwd, platform1c)
                prHelpers.updateInfobase(connString, admin1cUser, admin1cPwd, platform1c)
            }
        }
    }
}
