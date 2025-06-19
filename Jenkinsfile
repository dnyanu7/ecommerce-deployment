
// pipeline {
//   agent any

//   environment {
//     REMOTE_USER = 'ubuntu'
//     REMOTE_HOST = '192.168.1.131'               // <-- change this to your actual IP
//     REMOTE_DIR = '/home/ubuntu/ecommerce-app'
//     SSH_CREDENTIALS_ID = 'github-ssh-key'        // Jenkins credential ID for GitHub SSH key
//   }

//   stages {
//     stage('Clone Frontend Repo') {
//       steps {
//         dir('frontend') {
//           git branch: 'Ak_dev',
//               credentialsId: env.SSH_CREDENTIALS_ID,
//               url: 'git@github.com:abhi1231/navyaraga-ui.git'
//         }
//       }
//     }

//     stage('Clone Backend Repo') {
//       steps {
//         dir('backend') {
//           git branch: 'rudra_dev_cache',
//               credentialsId: env.SSH_CREDENTIALS_ID,
//               url: 'git@github.com:abhi1231/palmonas-reimagined-react.git'
//         }
//       }
//     }

//     stage('Build Frontend') {
//       steps {
//         dir('frontend') {
//           sh 'npm install'
//           sh 'npm run build'
//         }
//       }
//     }

//     stage('Build Backend') {
//       steps {
//         dir('backend/backend') {
//           sh './mvnw clean package -DskipTests || mvn clean package -DskipTests'
//         }
//       }
//     }

//     stage('Deploy to Server') {
//       steps {
//         sshagent (credentials: [env.SSH_CREDENTIALS_ID]) {
//           sh "ssh $REMOTE_USER@$REMOTE_HOST 'mkdir -p $REMOTE_DIR'"
//           sh "scp backend/target/*.jar $REMOTE_USER@$REMOTE_HOST:$REMOTE_DIR/app.jar"
//           // sh "scp -r frontend/build $REMOTE_USER@$REMOTE_HOST:$REMOTE_DIR/frontend"
//           sh "scp -r frontend/dist $REMOTE_USER@$REMOTE_HOST:$REMOTE_DIR/frontend"
//         }
//       }
//     }

//     stage('Restart Spring Boot App') {
//       steps {
//         sshagent (credentials: [env.SSH_CREDENTIALS_ID]) {
//           sh """
//             ssh $REMOTE_USER@$REMOTE_HOST '
//               pkill -f "java -jar" || true
//               nohup java -jar $REMOTE_DIR/app.jar > $REMOTE_DIR/logs.txt 2>&1 &
//             '
//           """
//         }
//       }
//     }
//   }

//   post {
//     success {
//       echo "✅ Deployment successful!"
//     }
//     failure {
//       echo "❌ Deployment failed!"
//     }
//   }
// }


pipeline {
  agent any

  tools {
    jdk 'jdk20'  // Name must match the JDK you added in Jenkins > Global Tool Configuration
  }

  environment {
    JAVA_HOME = "${tool 'jdk20'}"
    PATH = "${env.JAVA_HOME}/bin:${env.PATH}"

    REMOTE_USER = 'ubuntu'
    REMOTE_HOST = '192.168.1.123'
    REMOTE_DIR = '/home/ubuntu/ecommerce-app'
    SSH_CREDENTIALS_ID = 'github-ssh-key'
  }

  stages {
    stage('Check Java Version') {
      steps {
        sh 'java -version'
      }
    }

    stage('Clone Frontend Repo') {
      steps {
        dir('frontend') {
          git branch: 'Ak_dev',
              credentialsId: env.SSH_CREDENTIALS_ID,
              url: 'git@github.com:abhi1231/navyaraga-ui.git'
        }
      }
    }

    stage('Clone Backend Repo') {
      steps {
        dir('backend') {
          git branch: 'SK_Production',
              credentialsId: env.SSH_CREDENTIALS_ID,
              url: 'git@github.com:abhi1231/palmonas-reimagined-react.git'
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
        dir('backend/backend') {
          // Ensure JAVA_HOME is used
          sh 'java -version'
          sh 'mvn clean package -DskipTests'
        }
      }
    }
    stage('Debug: Show Jenkins Public Key') {
      steps {
        sshagent (credentials: [env.SSH_CREDENTIALS_ID]) {
          sh '''
            echo "----- Jenkins Public Key -----"
            ssh-add -L
            echo "------------------------------"
          '''
        }
      }
    }

    // stage('Deploy to Server') {
    //   steps {
    //     sshagent (credentials: [env.SSH_CREDENTIALS_ID]) {
    //       sh "ssh $REMOTE_USER@$REMOTE_HOST 'mkdir -p $REMOTE_DIR'"
    //       sh "scp backend/backend/target/*.jar $REMOTE_USER@$REMOTE_HOST:$REMOTE_DIR/app.jar"
    //       sh "scp -r frontend/dist $REMOTE_USER@$REMOTE_HOST:$REMOTE_DIR/frontend"
    //     }
    //   }
    // }
    
    stage('Deploy to Server') {
  steps {
    sshagent (credentials: [env.SSH_CREDENTIALS_ID]) {
      sh "ssh $REMOTE_USER@$REMOTE_HOST 'mkdir -p $REMOTE_DIR'"
      
      // Clear old frontend and backend artifacts
      sh "ssh $REMOTE_USER@$REMOTE_HOST 'rm -rf $REMOTE_DIR/frontend'"
      sh "ssh $REMOTE_USER@$REMOTE_HOST 'rm -f $REMOTE_DIR/app.jar'"

      // Deploy new backend jar
      sh "scp backend/backend/target/*.jar $REMOTE_USER@$REMOTE_HOST:$REMOTE_DIR/app.jar"
      
      // Deploy new frontend build
      sh "scp -r frontend/dist $REMOTE_USER@$REMOTE_HOST:$REMOTE_DIR/frontend"
    }
  }
}

    
    stage('Start Frontend Server') {
      steps {
        sshagent (credentials: [env.SSH_CREDENTIALS_ID]) {
          sh """
            ssh $REMOTE_USER@$REMOTE_HOST '
            
             sudo pkill -f "npx serve" || true
              nohup npx serve -s $REMOTE_DIR/frontend -l 3000 > $REMOTE_DIR/frontend-logs.txt 2>&1 &
            '
          """
        }
      }
    }

  //   stage('Restart Spring Boot App') {
  //     steps {
  //       sshagent (credentials: [env.SSH_CREDENTIALS_ID]) {
  //         sh """
  //           ssh $REMOTE_USER@$REMOTE_HOST '
  //             sudo pkill -f "java -jar" || true
  //             nohup java -jar $REMOTE_DIR/app.jar > $REMOTE_DIR/logs.txt 2>&1 &
  //           '
  //         """
  //       }
  //     }
  //   }
  // }
    
    stage('Restart Spring Boot App') {
  steps {
    sshagent (credentials: [env.SSH_CREDENTIALS_ID]) {
      sh """
        ssh $REMOTE_USER@$REMOTE_HOST '
          PID=\$(lsof -ti:8090)
          if [ ! -z "\$PID" ]; then
            echo "Killing process on port 8090: \$PID"
            kill -9 \$PID
          fi
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


