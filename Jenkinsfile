pipeline {
  agent any
  environment {
    DEV = 'http://192.168.100.3:8516'
    TEST = 'http://localhost:8081'
    TEST_THING = 'TestExample1'
    DOCKER_PATH = '/tmp/twdocker8_5'
    SETUP_DOCKER = 'false'
    INSTALL_EXTENSIONS = 'false'
    ADD_DEPENDECIES = 'true'
    INCLUDE_PASSWORDS = 'false'
    REMOVE_OLDPROJECT = 'false'
    PUBLISH_TO_SC = 'false'
    ARTIFACTID = 'costin-jenkins2'
    GROUPID = 'tw.jenkins'
    PACKAGE_VERSION = '1.0.0'
    MIN_PLATFORM_VERSION = '8.5.0'
    SCRIPT_PATH = '/tmp/Scripts/PipelineScripts'
    TEST_RESULT_PATH = '/tmp/test result'
    APPKEY = 'de367fd7-aca6-4b7d-adac-cfe218c430bb'
  }
  stages {
    stage('PackageApplication') {
      steps {
           sh '"${SCRIPT_PATH}"/packageApp.sh ${DEV} ${APPKEY} ${INSTALL_EXTENSIONS} "${SCRIPT_PATH}" ${ADD_DEPENDECIES} ${INCLUDE_PASSWORDS}'
           script{
            def resultPath = env.SCRIPT_PATH + "/packageapp.json"
            def parsedResult = readJSON file: resultPath
            def result = parsedResult.rows[0].result
            if (result.equals('Success')) {
                println(result)
            } else {
               error ("Error in Package Application stage " + result)
            }
            }
      }
    }
  stage('BuildDocker') {
      when {
          expression { return env.SETUP_DOCKER == 'true'}
      }
      steps {
           sh 'sudo docker-compose -f "${DOCKER_PATH}"/docker-compose-h2.yml up -d'
           sh 'cd "${DOCKER_PATH}"/twdevopssetup; sudo ./devOpsSetup.bash'
      }
    }
    stage('ImportApplication') {
      steps {
           sh '"${SCRIPT_PATH}"/importApp.sh ${TEST} ${APPKEY} ${INSTALL_EXTENSIONS} "${SCRIPT_PATH}" ${REMOVE_OLDPROJECT} ${INCLUDE_PASSWORDS}}'
           script{
            def resultPath = env.SCRIPT_PATH + "/importapp.json"
            def parsedResult = readJSON file: resultPath
            def result = parsedResult.rows[0].result
            if (result.equals('Success')) {
                println(result)
            } else {
               error ("Error in Package Application stage " + result)
            }
            }
      }
    }
    stage('ExecuteTests') {
      steps {
           sh '"${SCRIPT_PATH}"/executeTests.sh ${TEST} ${TEST_THING} ${APPKEY} "${TEST_RESULT_PATH}" ${PUBLISH_TO_SC} ${MIN_PLATFORM_VERSION} ${ARTIFACTID} ${GROUPID} ${PACKAGE_VERSION}'
           script{
                def resultPath = env.TEST_RESULT_PATH + "/inputs.json"
                def resultHtml = env.TEST_RESULT_PATH + "/index.html"
                def props = readJSON file: resultPath
                def htmlResult = props.rows[0].result
                if (fileExists(resultHtml)) {
                    new File(resultHtml).delete()
                }
                File f = new File(resultHtml)
                f.append(htmlResult)
           }
           publishHTML([allowMissing: false, alwaysLinkToLastBuild: true, keepAll: true, reportDir: env.TEST_RESULT_PATH, reportFiles: 'index.html', reportName: 'Results ThingWorx Tests', reportTitles: ''])
      }
    }

    }
  }
