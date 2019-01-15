pipeline {
  agent { dockerfile {
      filename 'Dockerfile'
      args '-v /home/sparklab/Datasets/euroc:/Datasets/euroc'
    }
  }
  stages {
    stage('Build') {
      steps {
        slackSend color: 'good', message: "Started Build <${env.BUILD_URL}|#${env.BUILD_NUMBER}> - Branch <${env.GIT_URL}|${env.GIT_BRANCH}>."
          cmakeBuild buildDir: 'build', buildType: 'Release', cleanBuild: false, generator: 'Unix Makefiles', installation: 'InSearchPath', sourceDir: '.', steps: [[args: '-j 8']]
      }
    }
    stage('Test') {
      steps {
        wrap([$class: 'Xvfb']) {
          sh 'cd build && make check -j 8'
            ctest arguments: '-T test --no-compress-output --verbose', installation: 'InSearchPath', workingDir: 'build/tests'
        }
      }
    }
    stage('Performance') {
      steps {
        wrap([$class: 'Xvfb']) {
          sh '/root/spark_vio_evaluation/evaluation/main_evaluation.py -r -a --save_plots --save_boxplots --save_results spark_vio_evaluation/experiments/euroc.yaml'
        }
      }
    }
  }
  post {
    always {
      echo 'Jenkins Finished'
      // Archive the CTest xml output
      archiveArtifacts (
          artifacts: 'build/tests/Testing/**/*.xml, spark_vio_evaluation/results/**/*.*',
          fingerprint: true
          )

      // Process the CTest xml output with the xUnit plugin
      xunit([CTest(
            deleteOutputFiles: true,
            failIfNotNew: true,
            pattern: 'build/tests/Testing/**/*.xml',
            skipNoTestFiles: false,
            stopProcessingIfError: true)
      ])

      // Clear the source and build dirs before next run
      deleteDir()
    }
    success {
      echo 'Success!'
      slackSend color: 'good', message: "Successful Build <${env.BUILD_URL}|#${env.BUILD_NUMBER}> - Branch ${env.GIT_BRANCH} finished in ${currentBuild.durationString}."
    }
    failure {
      echo 'Fail!'
      slackSend color: 'danger', message: "Failed - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
    }
    unstable {
      echo 'Unstable!'
      slackSend color: 'warning', message: "Unstable - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
    }
  }
}
