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

//notify build



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

    
node {
  properties([disableConcurrentBuilds()])
  ansiColor('xterm') {
  try {
    project = 'kube-dashboard'
    channel = '#huy.tran2'

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
            sh "aws ecr get-login --no-include-email --region us-east-1"
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
