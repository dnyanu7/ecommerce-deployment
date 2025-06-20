
pipeline {
  agent any

  tools {
    jdk 'jdk20'  // Name must match the JDK you added in Jenkins > Global Tool Configuration
  }

  environment {
    JAVA_HOME = "${tool 'jdk20'}"
    PATH = "${env.JAVA_HOME}/bin:${env.PATH}"

    REMOTE_USER = 'root'
    REMOTE_HOST = '103.174.102.148'
    REMOTE_DIR = '/home/jenkins/Navyaraaga'
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
          git branch: 'Dnyanesh_dev',
              credentialsId: env.SSH_CREDENTIALS_ID,
              url: 'git@github.com:abhi1231/navyaraga-ui.git'
        }
      }
    }

    stage('Clone Backend Repo') {
      steps {
        dir('backend') {
          git branch: 'dnyanesh_dev',
              credentialsId: env.SSH_CREDENTIALS_ID,
              url: 'git@github.com:abhi1231/palmonas-reimagined-react.git'
        }
      }
    }

    stage('Build Frontend') {
      steps {
        dir('frontend') {
          sh 'npm install'
          // sh 'node --max-old-space-size=2048 $(which npm) run build'
          sh 'node --max-old-space-size=2048 ./node_modules/vite/bin/vite.js build'

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

    
    // stage('Start Frontend Server') {
    //   steps {
    //     sshagent (credentials: [env.SSH_CREDENTIALS_ID]) {
    //       sh """
    //         ssh $REMOTE_USER@$REMOTE_HOST '
            
    //          sudo pkill -f "npx serve" || true
    //           // nohup npx serve -s $REMOTE_DIR/frontend -l 4000 > $REMOTE_DIR/frontend-logs.txt 2>&1 &
    //          setsid /snap/bin/serve -s frontend -l 4000 > frontend-logs.txt 2>&1 < /dev/null &
    //         '
    //       """
    //     }
    //   }
    // }

// stage('Start Frontend Server') {
//   steps {
//     sshagent (credentials: [env.SSH_CREDENTIALS_ID]) {
//       sh """
//         echo "🧹 Killing any running serve process..."
//         sudo pkill -f serve || true

//         echo "📁 Directory listing:"
//         ls -l /root/Navyaraaga/frontend || echo "❗ Frontend folder missing"

//         echo "🚀 COMMAND TO BE RUN:"
//         echo "/snap/bin/serve -d /root/Navyaraaga/frontend -l 4000 --no-ssl"

//         nohup /snap/bin/serve -d /root/Navyaraaga/frontend -l 4000 --no-ssl > /root/Navyaraaga/frontend-logs.txt 2>&1 < /dev/null &

//         sleep 3
//         echo "✅ Serve command executed"
//         pgrep -af serve || echo "❗ Serve process not found"
//       """
//     }
//   }
// }
    stage('Start Frontend Server') {
  steps {
    sshagent (credentials: [env.SSH_CREDENTIALS_ID]) {
      sh """
        ssh $REMOTE_USER@$REMOTE_HOST '
          echo "🧹 Killing existing serve process..."
          pkill -f "serve" || true

          echo "📂 Changing to app directory..."
          cd $REMOTE_DIR || exit 1

          echo "🚀 Starting frontend using serve..."
          setsid /snap/bin/serve -s frontend -l 4000 --no-ssl > frontend-logs.txt 2>&1 < /dev/null &
          
          sleep 3
          echo "✅ Serve launched on port 4000"
          pgrep -af serve || echo "❗ Serve not found"
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
          PID=\$(lsof -ti:8081)
          if [ ! -z "\$PID" ]; then
            echo "Killing process on port 8081: \$PID"
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

// pipeline {
//   agent any

//   tools {
//     jdk 'jdk20'
//   }

//   environment {
//     JAVA_HOME = "${tool 'jdk20'}"
//     PATH = "${env.JAVA_HOME}/bin:${env.PATH}"

//     REMOTE_USER = 'jenkins'
//     REMOTE_HOST = '103.174.102.148'
//     REMOTE_DIR = '/opt/navyaraaga'
//     SSH_CREDENTIALS_ID = 'github-ssh-key'
//   }

//   stages {
//     stage('Check Java Version') {
//       steps {
//         sh 'java -version'
//       }
//     }

//     stage('Clone Frontend Repo') {
//       steps {
//         dir('frontend') {
//           git branch: 'Dnyanesh_dev',
//               credentialsId: env.SSH_CREDENTIALS_ID,
//               url: 'git@github.com:abhi1231/navyaraga-ui.git'
//         }
//       }
//     }

//     stage('Clone Backend Repo') {
//       steps {
//         dir('backend') {
//           git branch: 'dnyanesh_dev',
//               credentialsId: env.SSH_CREDENTIALS_ID,
//               url: 'git@github.com:abhi1231/palmonas-reimagined-react.git'
//         }
//       }
//     }

//     stage('Build Frontend') {
//       steps {
//         dir('frontend') {
//           sh 'npm install'
//           sh 'node --max-old-space-size=2048 ./node_modules/vite/bin/vite.js build'
//         }
//       }
//     }

//     stage('Build Backend') {
//       steps {
//         dir('backend/backend') {
//           sh 'java -version'
//           sh 'mvn clean package -DskipTests'
//         }
//       }
//     }

//     stage('Deploy to Server') {
//       steps {
//         sshagent (credentials: [env.SSH_CREDENTIALS_ID]) {
//           sh "ssh $REMOTE_USER@$REMOTE_HOST 'mkdir -p $REMOTE_DIR'"
//           sh "ssh $REMOTE_USER@$REMOTE_HOST 'rm -rf $REMOTE_DIR/frontend'"
//           sh "ssh $REMOTE_USER@$REMOTE_HOST 'rm -f $REMOTE_DIR/app.jar'"
//           sh "scp backend/backend/target/*.jar $REMOTE_USER@$REMOTE_HOST:$REMOTE_DIR/app.jar"
//           sh "scp -r frontend/dist $REMOTE_USER@$REMOTE_HOST:$REMOTE_DIR/frontend"
//         }
//       }
//     }

//     stage('Start Frontend Server') {
//       steps {
//         sshagent (credentials: [env.SSH_CREDENTIALS_ID]) {
//           sh """
//             ssh $REMOTE_USER@$REMOTE_HOST '
//               echo "🧹 Killing any existing serve processes..."
//               pkill -f "serve" || true

//               echo "🚀 Starting serve on port 4000..."
//               cd $REMOTE_DIR || exit 1
//               setsid /snap/bin/serve -s frontend -l 4000 --no-ssl > frontend-logs.txt 2>&1 < /dev/null &

//               sleep 3
//               pgrep -af serve || echo "❗ Serve not found"
//             '
//           """
//         }
//       }
//     }

//     stage('Restart Spring Boot App') {
//       steps {
//         sshagent (credentials: [env.SSH_CREDENTIALS_ID]) {
//           sh """
//             ssh $REMOTE_USER@$REMOTE_HOST '
//               PID=\$(lsof -ti:8081)
//               if [ ! -z "\$PID" ]; then
//                 echo "Killing existing Spring Boot app: \$PID"
//                 kill -9 \$PID
//               fi

//               echo "🚀 Starting Spring Boot app..."
//               nohup java -jar $REMOTE_DIR/app.jar > $REMOTE_DIR/backend-logs.txt 2>&1 &
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

