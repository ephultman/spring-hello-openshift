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
    stage('Create Dev Configs') {
          when {
            not {
              expression {
                openshift.withCluster() {
                  openshift.withProject(DEV_ENV) {
                    return openshift.selector("bc", appName).exists()
                  }
                }
              }
            }
          }
          steps {
            script {
                openshift.withCluster() {
                    openshift.withProject(DEV_ENV) {
                      def app = openshift.newApp("${S2I_IMAGE}~${REPO_URL}", "--name='${appName}'", "--strategy=source")
                      sleep 5
                      def logs = app.narrow('bc').logs('-f')
                    }
                }
            }
          }
        }
    stage('Start Build') {
      steps {
        script {
            openshift.withCluster() {
                openshift.withProject(DEV_ENV) {
                    def buildSelector = openshift.startBuild(appName).narrow('bc')
                    sleep 5
                    def logs = buildSelector.logs('-f')
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
