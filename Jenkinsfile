pipeline {
  agent any
  options { skipDefaultCheckout() }
  parameters {
    string name: 'BRANCH', defaultValue: 'master', description: 'Which branch do you want to build?'
    booleanParam name: 'PUBLISH', defaultValue: true, description: 'Publish the build?'
  }
  tools {
    dotnetsdk 'Core 3.1'
  }
  stages {
    stage ('Build') {
      steps {
        checkout([$class: 'GitSCM',
            branches: [[name: '*/main']],
            doGenerateSubmoduleConfigurations: false,
            extensions: [
              [
                $class: 'SparseCheckoutPaths',
                sparseCheckoutPaths: [
                  [ $class: 'SparseCheckoutPath', path: 'csharp/unit-testing/' ]]
                ]
              ],
            submoduleCfg: [],
            userRemoteConfigs: [[url: 'git@github.com:balakine/samples.git', refspec: "+refs/heads/${params.BRANCH}:refs/remotes/origin/${params.BRANCH}"]]
        ])
        bat "git status"
        bat "git branch -a"
//        bat "git fetch --all"
//        bat "git branch -a"
//        bat "git checkout remotes/origin/${params.BRANCH}"
        dotnetBuild()
      }
      post {
        unsuccessful {
          error 'Build failed'
        }
      }
    }
    stage('Parallel tests') {
      matrix {
        axes {
          axis {
            name 'PROJECT'
            values 'MSTest.Project', 'NUnit.TestProject', 'XUnit.TestProject'
          }
        }
        stages {
          stage ('Test') {
            steps {
              dir("${PROJECT}") {
                dotnetTest logger: 'trx', resultsDirectory: '.'
              }
            }
          }
        }
      }
      post {
        always {
          archiveArtifacts artifacts: '*//*.trx'
        }
        unsuccessful {
          error 'Test failed'
        }
      }
    }
    stage ('Publish') {
      when { expression { params.PUBLISH } }
      steps {
        dotnetPublish outputDirectory: 'my-publish', noBuild: true
        zip zipFile: 'project.zip', dir: 'my-publish', archive: true
      }
    }
  }
  post {
    cleanup {
      cleanWs()
    }
  }
}
