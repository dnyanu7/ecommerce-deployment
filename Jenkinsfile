pipeline {
  agent any

  environment {
    REMOTE_USER = 'ubuntu'
    REMOTE_HOST = '192.168.1.131'               // <-- change this to your actual IP
    REMOTE_DIR = '/home/ubuntu/ecommerce-app'
    SSH_CREDENTIALS_ID = 'github-ssh-key'        // Jenkins credential ID for GitHub SSH key
  }

  stages {
    stage('Clone Frontend Repo') {
      steps {
        dir('frontend') {
          sshagent (credentials: [env.SSH_CREDENTIALS_ID]) {
            git url: 'git@github.com:abhi1231/navyaraga-ui.git', branch: 'main'
          }
        }
      }
    }

    stage('Clone Backend Repo') {
      steps {
        dir('backend') {
          sshagent (credentials: [env.SSH_CREDENTIALS_ID]) {
            git url: 'git@github.com:abhi1231/palmonas-reimagined-react.git', branch: 'main'
          }
        }
      }
    }

    stage('Build Frontend') {
      steps {
        dir('frontend') {
          sh 'npm install'
          sh 'npm run build'
        }
      }
    }

    stage('Build Backend') {
      steps {
        dir('backend') {
          sh './mvnw clean package -DskipTests || mvn clean package -DskipTests'
        }
      }
    }

    stage('Deploy to Server') {
      steps {
        sshagent (credentials: [env.SSH_CREDENTIALS_ID]) {
          sh "ssh $REMOTE_USER@$REMOTE_HOST 'mkdir -p $REMOTE_DIR'"
          sh "scp backend/target/*.jar $REMOTE_USER@$REMOTE_HOST:$REMOTE_DIR/app.jar"
          sh "scp -r frontend/build $REMOTE_USER@$REMOTE_HOST:$REMOTE_DIR/frontend"
        }
      }
    }

    stage('Restart Spring Boot App') {
      steps {
        sshagent (credentials: [env.SSH_CREDENTIALS_ID]) {
          sh """
            ssh $REMOTE_USER@$REMOTE_HOST '
              pkill -f "java -jar" || true
              nohup java -jar $REMOTE_DIR/app.jar > $REMOTE_DIR/logs.txt 2>&1 &
            '
          """
        }
      }
    }
  }

  post {
    success {
      echo "✅ Deployment successful!"
    }
    failure {
      echo "❌ Deployment failed!"
    }
  }
}

