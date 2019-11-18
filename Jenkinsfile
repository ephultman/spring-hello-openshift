def appName = 'spring-hello'
def DEV_ENV = 'development'
def S2I_IMAGE = 'fabric8/s2i-java'
def REPO_URL = 'https://github.com/ephultman/spring-hello-openshift'

pipeline {
  agent any
  options {
    timeout(time: 20, unit: 'MINUTES')
  }
  stages {
    stage('Cleanup') {
          steps {
            script {
                openshift.withCluster() {
                    openshift.withProject(DEV_ENV) {
                      openshift.selector("all", [ app : appName ]).delete()
                    }
                }
            }
          }
        }
    stage('Create/Poke Build Config') {
      steps {
        script {
            openshift.withCluster() {
                openshift.withProject(DEV_ENV) {
                  if (openshift.selector("bc", appName).exists()) {
                    def buildSelector = openshift.startBuild(appName).narrow('bc')
                    sleep 3
                    def logs = buildSelector.logs('-f')
                  } else {
                    def app = openshift.newApp("${S2I_IMAGE}~${REPO_URL}", "--name='${appName}'", "--strategy=source")
                    sleep 3
                    def bc = app.narrow('bc')
                    def logs = bc.logs('-f')
                    }
               }
            }
        }
      }
    }
    stage('Confirm Build') {
      steps {
        script {
            openshift.withCluster() {
                openshift.withProject(DEV_ENV) {
                  def bc = openshift.selector("bc", appName)
                  def builds = bc.related('builds')
                  timeout(5) {
                    builds.untilEach(1) {
                      return (it.object().status.phase == "Complete")
                    }
                  }
                }
            }
        }
      }
    }
    stage('Deploy') {
      steps {
        script {
            openshift.withCluster() {
                openshift.withProject(DEV_ENV) {
                  def rm = openshift.selector("dc", appName).rollout()
                  timeout(5) {
                    openshift.selector("dc", appName).related('pods').untilEach(1) {
                      return (it.object().status.phase == "Running")
                    }
                  }
                }
            }
        }
      }
    }
    stage('Tag Image') {
      steps {
        script {
            openshift.withCluster() {
                openshift.withProject(DEV_ENV) {
                  openshift.tag("${appName}:latest", "${appName}-dev:latest")
                }
            }
        }
      }
    }
  }
}
