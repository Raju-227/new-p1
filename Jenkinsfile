pipeline {
  agent any
  options { timestamps() }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Archive HTML') {
      steps {
        // Keeps a copy of the site as a Jenkins artifact
        archiveArtifacts artifacts: 'index.html', fingerprint: true
      }
    }

    stage('Docker Build & Push') {
      steps {
        script {
          def tag = env.BUILD_NUMBER
          withCredentials([usernamePassword(credentialsId: 'docker-hub',
                                           usernameVariable: 'DH_USER',
                                           passwordVariable: 'DH_PASS')]) {
            sh """
              set -eux
              echo "\$DH_PASS" | docker login -u "\$DH_USER" --password-stdin
              docker build -t "\$DH_USER/memento-web:${tag}" -t "\$DH_USER/memento-web:latest" .
              docker push "\$DH_USER/memento-web:${tag}"
              docker push "\$DH_USER/memento-web:latest"
            """
            // Expose for next stages
            env.IMAGE_REPO = "${DH_USER}/memento-web"
            env.IMAGE_TAG  = tag
          }
        }
      }
    }
     stage('Deploy to Docker Swarm') {
       steps {
         script {
           withCredentials([usernamePassword(credentialsId: 'docker-hub',
                                            usernameVariable: 'DH_USER',
                                            passwordVariable: 'DH_PASS')]) {
             sh """
               set -eux
               echo "\$DH_PASS" | docker login -u "\$DH_USER" --password-stdin
               docker swarm init 2>/dev/null || true
               docker service create --name memento-web --replicas 2 -p 8098:80 \\
                  "\$DH_USER/memento-web:${IMAGE_TAG}" || \\
               docker service update --image "\$DH_USER/memento-web:${IMAGE_TAG}" memento-web
           """
           }
         }
       }
     }
  }

  post {
    always { sh 'docker logout || true' }
  }
}
