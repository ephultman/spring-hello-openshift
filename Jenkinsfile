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
    stage('Create Build Config') {
      steps {
        script {
            openshift.withCluster() {
                openshift.withProject() {
                  if !openshift.selector("bc", appName).exists() {
                    openshift.newApp("https://github.com/ephultman/spring-hello-openshift", "--name='${appName}'")
                    }
               }
            }
        }
      }
    }
    stage('build') {
      steps {
        script {
            openshift.withCluster() {
                openshift.withProject() {
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
    stage('deploy') {
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
    stage('tag') {
      steps {
        script {
            openshift.withCluster() {
                openshift.withProject() {
                  openshift.tag("${appName}:latest")
                }
            }
        }
      }
    }
  }
}