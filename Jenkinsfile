#!/usr/bin/env groovy

imageName= ''

//build image
def buildImage(String imageName='', String dockerFile='Dockerfile', String location='.', String dockerRepo='764574898083.dkr.ecr.us-east-1.amazonaws.com') {
    sh """
        egrep -q '^FROM .* (AS|as) builder\$' ${dockerFile} \
            && docker build -t ${dockerRepo}/${imageName}-stage-builder --target builder -f ${dockerFile} ${location}
        docker build -t ${dockerRepo}/${imageName} -f ${dockerFile} ${location}
    """
}

//push image
def pushImage(String imageName='', String dockerRepo='764574898083.dkr.ecr.us-east-1.amazonaws.com') {
	// imageName: ${prj_name}:${img_tag}, img_tag: env.BRANCH_NAME
	buildNumber = env.BUILD_NUMBER
    imageBuild = "${imageName}-build-${buildNumber}"
    sh "docker tag ${dockerRepo}/${imageName} ${dockerRepo}/${imageBuild}"
    sh "docker push ${dockerRepo}/${imageBuild}"
}

//checkout git
def checkoutGit(scm) {
	scmVars = checkout([
            $class: 'GitSCM',
            branches: scm.branches,
            doGenerateSubmoduleConfigurations: scm.doGenerateSubmoduleConfigurations,
            extensions: scm.extensions + [
                [$class: 'LocalBranch'],
                [$class: 'CloneOption', depth: 2, noTags: true, reference: '', shallow: true]
            ],
            userRemoteConfigs: scm.userRemoteConfigs
        ])
    return scmVars
}

channel = ''

def notifyBuild(String buildStatus = 'STARTED', String channel = '#') {
  // build status of null means successful
  buildStatus =  buildStatus ?: 'SUCCESSFUL'

  // Default values
  def colorName = 'RED'
  def colorCode = '#FF0000'
  def subject = "${buildStatus} >> <${env.RUN_DISPLAY_URL} | *${env.JOB_NAME}* [build-${env.BUILD_NUMBER}]>"
  def summary = "${subject}\n[<${env.RUN_CHANGES_DISPLAY_URL} | changes-log >]"


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

  slackSend (channel: channel, color: colorCode, message: summary)
}

    
node {
  properties([disableConcurrentBuilds()])
  ansiColor('xterm') {
  try {
    project = 'test-jenkins'
    channel = '#ai-k8s-ci'

    notifyBuild('STARTED', channel)

    stage('pull code') {
      scmVars = checkoutGit(scm)
    }
    
    // remove potential '/' in branch name (Ex: feature/bla_bla)
    branch = env.BRANCH_NAME.replaceAll(/\//, '_')
    buildNumber = env.BUILD_NUMBER
    imageBranch = "${project}:${branch}"
    imageBuild = "${project}:${branch}-build-${buildNumber}"

    switch(env.BRANCH_NAME) {  
      case 'dev':

        stage ('login ECR') {
            sh "\$(aws ecr get-login --no-include-email --region us-east-1)"
        }

        stage('img -> dockerhub') {
          buildImage(imageBranch)
          pushImage(imageBranch)
        }
      break
    }
  } catch (e) {
    // exception thrown, build failed
    currentBuild.result = "FAILED"
    throw e
  } finally {
    // success or failure, always send notifications
  }
  }
}
