#!groovy

node {

    step([$class: 'WsCleanup'])

    stage "Checkout Git repo"
        checkout scm

    stage "Run test"
        sh "docker run -v \$(pwd):/data --rm usemtech/nodejs-mocha npm install"
        sh "docker run -v \$(pwd):/data --rm usemtech/nodejs-mocha npm install chai chai-http"
        sh "docker run -v \$(pwd):/data --rm usemtech/nodejs-mocha npm test"
    stage "Build RPM"
        sh "[ -d ./rpm] || mkdir ./rpm"
        sh "docker run -v \$(pwd):/data/aik-app -v \$(pwd)/rpm:/data/rpm --rm tenzer/fpm -s dir -t rpm -n aik-app -v \$(git rev-parse --short HEAD) --description \"Aik app\" --directories /var/www/aik-app --package /data/rpm/aik-app-\$(git rev-parse --short HEAD).rpm /data/aik-app=/var/www/"
    stage "Update YUM repo"
        sh "[ -d ~/repo/rpm/aik-app] || mkdir -p ~/repo/rpm/aik-app/"
        sh "sudo mv ./rpm/*.rpm ~/repo/rpm/aik-app/"
        sh "createrepo ~/repo/"
        sh "aws s3 sync ~/repo s3://brz-bucket-automatizacion/ --region us-west-2 --delete"
    stage "Check YUM repo"
        sh "sudo yum clean all"
        sh "sudo yum update -y"
        sh "sudo yum info aik-app-\$(git rev-parse --short HEAD)"
    stage "Trigger downstream"

        def versionApp = sh returnStdout: true, script:"printf \$(git rev-parse --short HEAD)"
        build job: "frontend_pipeline_CD", parameters: [[$class: "StringParameterValue", name: "APP_VERSION", value: "${versionApp}-1"]], wait: false

}
