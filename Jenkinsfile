import groovy.transform.Field

jsl = library(
  identifier: "jenkins-shared-library@1.0.57",
  retriever: modernSCM(
    [
      $class: 'GitSCMSource',
      remote: 'https://github.com/ACRCode/jenkins-shared-library.git',
      credentialsId: 'github'
    ]
  )
)

// -----------------------------------------
// the following variables are customizable
// -----------------------------------------
@Field
def appName = 'isopod'

@Field
def versionPrefix = '1.0'

// -----------------------------------------
//      Do not change the values below
// -----------------------------------------
@Field
def project = 'acrcode'

@Field
def image = "${project}/${appName}"

def getImageTag(){
  return "${image}:${version.buildVersion}"
}

@Field
def LAST_STAGE


version = jsl.org.acr.jenkins.Version.new(this, versionPrefix)
errorSummary = jsl.org.acr.jenkins.ErrorSummary.new(this)
slack = jsl.org.acr.jenkins.Slack.new(this, 'slack-pipeline-token', 'SLACK_LEGACY_TOKEN')


pipeline{
  agent {
    kubernetes {
      label "${appName}-build-pod"
      yamlFile 'jenkins/executor.yaml'
    }
  }

  environment{
    DOCKER_CREDS = credentials('docker-hub')
  }

  stages{
    stage('Checkout'){
      steps{
        script{LAST_STAGE = env.STAGE_NAME}
        checkout scm

        script{version.changeDisplayNameToBuildVersion()}
      }
    }


    stage('Build and Publish Image to DockerHub'){
      steps{
        container(name: 'kaniko', shell: '/busybox/sh') {
          withEnv(['PATH+EXTRA=/busybox']) {
            sh """#!/busybox/sh
            /kaniko/executor --context `pwd` --destination ${imageTag}
            """
          }
        }
      }
    }
  }

  post{
    success {
      script{slack.notifySuccess('dmd', appName, version.buildVersion)}
    }
    failure {
      script{errorSummary.generate(LAST_STAGE)}

      script{slack.notifyFailure('dmd', appName, version.buildVersion, LAST_STAGE)}
    }
  }
}
