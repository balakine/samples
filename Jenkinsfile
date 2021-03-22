pipeline {
  agent any
  stages {
    stage('Build') {
      steps {
        bat 'git checkout remotes/origin/${params.BRANCH}'
        dotnetBuild()
      }
    }

  }
}