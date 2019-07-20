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

      stage "Checkout"
        checkout scm
        checkout([$class: 'GitSCM', branches: [[name: '*/openwrt-19.07']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/openwrt/openwrt.git']]])
        sh "ls -lah"
      
      stage "Feeds"
        sh "rm feeds.conf"
        sh "wget https://raw.githubusercontent.com/benlue-org/openwrt-builder/master/feeds/feeds.conf"
        sh label: 'Feeds Update', returnStdout: true, script: './scripts/feeds update -a'
        sh label: 'Feeds Install', returnStdout: true, script: './scripts/feeds install -a'
        sh "rm .config"
        sh "wget https://raw.githubusercontent.com/benlue-org/openwrt-builder/master/profiles/ar71xx/tlwdr4300v1/diffconfig"
        sh "mv diffconfig .config"
        sh "echo CONFIG_TARGET_ar71xx_generic_DEVICE_tl-wdr4300-v1=y"
        sh "make defconfig"
      
      stage "Build"
        sh "make clean"
        sh "make -j4 V=s"

      stage "Archive"
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
