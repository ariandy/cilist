pipeline {
  agent any

  environment {
    GIT_COMMIT_SHORT = sh(returnStdout: true, script: '''echo $GIT_COMMIT | head -c 7''')
  }

  stages {
    stage('Prepare .env') {
      steps {
        sh 'echo GIT_COMMIT_SHORT=$(echo $GIT_COMMIT_SHORT) > .env'
      }
    }

    stage('Build database') {
      steps {
        dir('database') {
          sh 'docker build . -t cilist-pipeline-db:$GIT_COMMIT_SHORT'
          sh 'docker tag cilist-pipeline-db:$GIT_COMMIT_SHORT ooxyz/cilist-pipeline-db:$GIT_COMMIT_SHORT'
          sh 'docker push ooxyz/cilist-pipeline-db:$GIT_COMMIT_SHORT'
        }
      }
    }

    stage('Build backend') {
      steps {
        dir('backend') {
          sh 'docker build . -t cilist-pipeline-be:$GIT_COMMIT_SHORT'
          sh 'docker tag cilist-pipeline-be:$GIT_COMMIT_SHORT ooxyz/cilist-pipeline-be:$GIT_COMMIT_SHORT'
          sh 'docker push ooxyz/cilist-pipeline-be:$GIT_COMMIT_SHORT'
        }
      }
    }

    stage('Build frontend') {
      steps {
        dir('frontend') {
          sh 'docker build . -t cilist-pipeline-fe:$GIT_COMMIT_SHORT'
          sh 'docker tag cilist-pipeline-fe:$GIT_COMMIT_SHORT ooxyz/cilist-pipeline-fe:$GIT_COMMIT_SHORT'
          sh 'docker push ooxyz/cilist-pipeline-fe:$GIT_COMMIT_SHORT'
        }
      }
    }

    stage('Deploy to remote server') {
      steps {
        sshPublisher(publishers: [sshPublisherDesc(configName: 'Jenkins-Remote', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '''docker compose up -d

        sleep 40

        docker compose restart backend''', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '.env,docker-compose.yaml')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
      }
    }
  }
}