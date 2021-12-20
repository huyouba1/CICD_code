node ('ydzs-jnlp'){
    stage('Prepare') {
        echo "1.Prepare Stage"
        checkout scm
        script {
            build_tag = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
            if (env.BRANCH_NAME != 'master') {
                build_tag = "${env.BRANCH_NAME}-${build_tag}"
            }
        }
    }
  stage('Test') {
    echo "2. Test Stage"
  }
  stage('Build') {
    echo "3. Build Stage"
    sh "docker build -t iregistry.baidu-int.com/library/jenkins-demo:${build_tag} ."
  }
  stage('Push') {
    echo "4. Push Docker Image Stage"
    withCredentials([usernamePassword(credentialsId:  'dockerAuth',usernameVariable: 'dockerHubUser', passwordVariable: 'dockerHubPassword')]) {
      sh "docker login  iregistry.baidu-int.com -u ${dockerHubUser} -p ${dockerHubPassword}"
      sh "docker push iregistry.baidu-int.com/library/jenkins-demo:${build_tag}"
    }
    
  }
  stage('YAML') {
    echo "5. Change YAML FILE Stage"
    def userInput = input(
      id: 'userInput',
      message: 'Choose a deploy env',
      parameters: [
        [
          $class: 'ChoiceParameterDefinition',
          choices: 'Dev\nQA\nProd',
          name: "Env"
        ]
      ]
    )
    echo "This is a deploy step to ${userInput} env"
    sh "sed -i  's#<BUILD_TAG>#${build_tag}#' k8s.yaml"
  }
  stage('Deploy') {
    echo "6. Deploy Stage"
    sh "kubectl apply -f k8s.yaml --record"
  }
}
