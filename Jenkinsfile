
def appName = 'isopod'

def buildVersion(){
  def retval = "${versionPrefix}.${gitVersion}"
  if (env.BRANCH_NAME != 'master') {
    retval + "-${env.BRANCH_NAME}"
  }
  return retval
}

def imageTag(){
  return "${image}:${buildVersion()}"
}

pipeline{
  agent {
    kubernetes {
      label "${appName}-build-pod"
      yamlFile 'jenkins/executor.yaml'
    }
  }

  environment{
    // -----------------------------------------
    // the following variables are customizable
    // -----------------------------------------
    versionPrefix = '0.1'

    // -----------------------------------------
    //      Do not change the values below
    // -----------------------------------------
    project = 'acrcode'
    gitVersion = 'notset'
    image = "${project}/${appName}"

    DOCKER_CREDS = credentials('docker-hub')
  }

  stages{
    stage('Checkout'){
      steps{
        checkout scm

        script{
          gitVersion = sh(
            script: 'git rev-list --no-merges --count $(git rev-parse HEAD) -- .',
            returnStdout: true
          ).trim()

          currentBuild.displayName = buildVersion()
        }
      }
    }


    stage('Build and Publish Image to DockerHub'){
      steps{
        container(name: 'kaniko', shell: '/busybox/sh') {
          withEnv(['PATH+EXTRA=/busybox']) {
            sh """#!/busybox/sh
            /kaniko/executor --context `pwd` --destination ${imageTag()}
            """
          }
        }
      }
    }
  }
}
