stage 'CI'
node {

    checkout scm

    // pull dependencies from npm
    // on windows use: bat 'npm install'
    sh 'npm install'

    // stash code & dependencies to expedite subsequent testing
    // and ensure same code & dependencies are used throughout the pipeline
    // stash is a temporary archive
    stash name: 'everything', 
          excludes: 'test-results/**', 
          includes: '**'
    
    // test with PhantomJS for "fast" "generic" results
    // on windows use: bat 'npm run test-single-run -- --browsers PhantomJS'
    sh 'npm run test-single-run -- --browsers PhantomJS'
    
    // archive karma test results (karma is configured to export junit xml files)
    step([$class: 'JUnitResultArchiver', testResults: 'test-results/**/test-results.xml'])
          
}

stage('Browser Testing') {
     parallel chrome: {
        runTest('Chrome')
    }, PhantomJS: {
        runTest('PhantomJS')
    }
}

stage('deploy'){
    node {
        sh 'docker-compose up -d --build'
        notify('Solitaire Deployed!')
    }
}

def runTest(browser) {
    node {
        sh 'rm -rf *'
        unstash 'everything'
        sh "npm run test-single-run -- --browsers ${browser}"
        step([$class: 'JUnitResultArchiver', testResults: 'test-results/**/test-results.xml'])
    }
}

def notify(status){
    emailext (
      to: "a.cassianoweber@gmail.com@gmail.com",
      subject: "${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
      body: """<p>${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        <p>Check console output at <a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a></p>""",
    )
}
