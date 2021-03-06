node {
    
    stage('repository_pull') {
      // Checkout the master branch of the Laravel framework repository
      sh 'env | sort'
      git branch: 'master', url: 'https://github.com/ashabdullah/simple-jenkins.git'
    }
    
    stage("composer_install") {
      // Run `composer update` as a shell script
      // sh 'mysqladmin -uroot -p$UNITTEST_MYSQL_ROOT_PW create eoswallet'
      // sh 'mysqladmin -uroot -p$UNITTEST_MYSQL_ROOT_PW create eos_wallet_test'
      sh 'composer install'
      sh 'cp .env.example .env'
      sh 'php artisan key:generate'
      sh 'php artisan config:clear'
      sh 'php artisan config:cache'
      sh 'mkdir -p storage/{sessions,views,cache,framework/{views,sessions}}'
      // sh 'php artisan vendor:publish'
      //sh 'php artisan db:seed'
      //sh 'echo | php artisan passport:install -qn'
    }
    
    stage("phpunit") {
      // Run PHPUnit
      sh 'vendor/bin/phpunit'

      // If this is the master or develop branch being built then run
      // additional integration tests
      if (["master", "develop"].contains(env.NODE_NAME)) {
          stage("integration_tests") {
              // Execute some scripts here
              // sh ''
          }
      }
      
    }
    
    stage("zip_archive") {
      // Zip the archive to prepare it for an S3 upload
      sh "zip -r ${env.BUILD_TAG}.zip ."
    }
    
    stage("s3_upload") {
      // Upload deployment assets to S3
      sh "aws s3 cp ${env.BUILD_TAG}.zip s3://jenkinscodedeploy-codedeploybucket-1syidevv1kpoy --region us-east-1"
    }
    
    stage('Deploy approval'){
    input "Deploy to prod?"
    }    
    stage("codedeploy_execute") {
      // Deploy assets
      switch (env.NODE_NAME) {
        case "master":
          stage("codedeploy") {
            // Push zip to CodeDeploy
            sh "aws deploy push --application-name JenkinsCodeDeploy-DemoApplication-D99GG6EWTYJY --s3-location s3://jenkinscodedeploy-codedeploybucket-1syidevv1kpoy/${env.BUILD_TAG}.zip --region us-east-1 --source ./"
            // Deploy zip
            sh "aws deploy create-deployment --application-name JenkinsCodeDeploy-DemoApplication-D99GG6EWTYJY --s3-location bucket=jenkinscodedeploy-codedeploybucket-1syidevv1kpoy,key=${env.BUILD_TAG}.zip,bundleType=zip --deployment-group-name JenkinsCodeDeploy-DemoFleet-FTZ157ONXELL --deployment-config-name CodeDeployDefault.OneAtATime --description 'Deploying App Build: ${env.BUILD_NUMBER}' --region us-east-1"
            }
        break
        case "develop":
          stage("codedeploy") {
            // Deploy to the test server
            }
        break
        default:
        // No deployments for other branches
        break
      }
    }

    stage('cleanup') {
    // Recursively delete all files and folders in the workspace
    // using the built-in pipeline command
    deleteDir()
    }
}
