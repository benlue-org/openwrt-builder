def _pipelineNotify(String buildStatus = 'STARTED') {
    // build status of null means successful
    buildStatus =  buildStatus ?: 'SUCCESSFUL'

    // Default values
    def colorName = 'RED'
    def colorCode = '#FF0000'
    def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
    def summary = "${subject} (${env.BUILD_URL})"

    // Override default values based on build status
    if (buildStatus == 'STARTED') {
        color = 'YELLOW'
        colorCode = '#FFFF00'
    } else if (buildStatus == 'SUCCESSFUL') {
        color = 'GREEN'
        colorCode = '#00FF00'
    } else {
        color = 'RED'
        colorCode = '#FF0000'
    }
}

node {
  try {
      _pipelineNotify()

      stage 'Checkout'
      checkout scm
      checkout([$class: 'GitSCM', branches: [[name: '*/openwrt-19.07']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/openwrt/openwrt.git']]])
      
      stage 'Grab Feeds'
      sh "cd openwrt"
      sh "./scripts/feeds update -a"
      
      stage 'Install Feeds'
      sh "./scripts/feeds install -a"
      
      stage 'Build'
      sh "make V=s"

      stage 'Archive'
      archive 'output/**/*'
  }
  catch (e) {
      currentBuild.result = "FAILED"
      throw e
  }
  finally {
      _pipelineNotify(currentBuild.result)
  }
}
