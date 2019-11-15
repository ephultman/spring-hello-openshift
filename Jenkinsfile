def appName = 'spring-hello'
pipeline {
  agent any
  options {
    timeout(time: 20, unit: 'MINUTES')
  }
  stages {
    stage('Preamble') {
        steps {
            script {
                openshift.withCluster() {
                    openshift.withProject() {
                        echo "Using project: ${openshift.project()}"
                    }
                }
            }
        }
    }
    stage('Cleanup') {
          steps {
            script {
                openshift.withCluster() {
                    openshift.withProject() {
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
                openshift.withProject() {
 //                 if (openshift.selector("bc", appName).exists()) {
 //                   def buildSelector = openshift.startBuild(appName).narrow('bc')
 //                   def logs = buildSelector.logs('-f')
 //                 } else {
                    def app = openshift.newApp("fabric8/s2i-java~https://github.com/ephultman/spring-hello-openshift", "--name='${appName}'", "--strategy=source")
                    sleep 5
                    def bc = app.narrow('bc')
                    def logs = bc.logs('-f')
 //                   }
               }
            }
        }
      }
    }
    stage('Confirm Build') {
      steps {
        script {
            openshift.withCluster() {
                openshift.withProject() {
                  def bc = openshift.selector("bc", appName)
                  def builds = openshift.selector("bc", appName).related('builds')
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
                openshift.withProject() {
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
                openshift.withProject() {
                  openshift.tag("${appName}:latest", "${appName}-dev:latest")
                }
            }
        }
      }
    }
  }
}