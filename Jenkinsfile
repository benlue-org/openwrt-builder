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

properties([parameters([choice(choices: ['TL-WDR4300v1', 'TL-MR3020v1', 'TL-WR1043NDv1'], description: 'Select your device to build', name: 'device'), 
						choice(choices: ['master', 'openwrt-18.06', 'openwrt-19.07', 'develop'], description: 'Select your Branch to buils', name: 'branch'), 
						choice(choices: ['test1', 'test2', 'test3', 'test4'], description: 'Treffe deine Auswahl für Build 3', name: 'test'),
						booleanParam(defaultValue: true, description: 'Willst du ein clean build?', name: 'make clean')]), [$class: 'JiraProjectProperty'], pipelineTriggers([[$class: 'PeriodicFolderTrigger', interval: '1d']
						])
])

node {
  try {
      _pipelineNotify()


      stage('SCM Checkout') {
        echo "Pulling changes from branch ${params.branch}"
        checkout([$class: 'GitSCM',
                  branches: [[name: "${params.branch}"]],
                  doGenerateSubmoduleConfigurations: false,
                  extensions: [[$class: 'CleanBeforeCheckout']],
                  submoduleCfg: [],
                  userRemoteConfigs: [[url: 'https://github.com/openwrt/openwrt.git']]
                ])  
          
        //git url: 'https://github.com/openwrt/openwrt.git', branch "${params.branch}"
        //checkout scm
        //checkout([$class: 'GitSCM', branches: [[name: '*/openwrt-18.06']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/openwrt/openwrt.git']]])
        sh "ls -lah"
      }
      
      stage('Feeds') {
        //sh "mv feeds.conf feeds.conf.old"
        sh "wget https://raw.githubusercontent.com/benlue-org/openwrt-builder/master/feeds/feeds.conf"
        sh label: 'Feeds Update', returnStdout: true, script: './scripts/feeds update -a'
        sh label: 'Feeds Install', returnStdout: true, script: './scripts/feeds install -a'
        sh "rm -f .config"
        sh "rm -f diffconfig"
        sh "wget https://raw.githubusercontent.com/benlue-org/openwrt-builder/master/profiles/ar71xx/tlwdr4300v1/diffconfig"
        sh "mv diffconfig .config"
        sh "echo CONFIG_TARGET_ar71xx_generic_DEVICE_tl-wdr4300-v1=y"
        sh "make defconfig"
      }
      
      stage('Build') {
        //sh "make clean"
        sh label: 'Make Clean', returnStdout: true, script: 'make clean'
	sh label: 'Make Info', returnStdout: true, script: 'make info'      
        sh label: 'Build Process', returnStdout: true, script: 'make -j1 V=s'
        //sh "make -j4 V=s"
      }
      
      stage('Archive') {
        archiveArtifacts 'bin/targets/**/**/*.bin'
      }
  }
  catch (e) {
      currentBuild.result = "FAILED"
      throw e
  }
  finally {
      _pipelineNotify(currentBuild.result)
  }
}
