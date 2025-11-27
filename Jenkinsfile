pipeline {
  agent {
    docker {
      image 'python:3.11'
      args '-u root:root'   // run as root so venv/pip work
    }
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Build') {
      steps {
        sh '''
          python -m venv venv
          . venv/bin/activate
          pip install --upgrade pip
          pip install -r requirements.txt
        '''
      }
    }

    stage('Test') {
      steps {
        sh '''
          . venv/bin/activate
          python -m unittest discover -s .
        '''
      }
    }

    stage('Deploy') {
      steps {
        sh '''
          mkdir -p ${WORKSPACE}/python-app-deploy
          cp ${WORKSPACE}/app.py ${WORKSPACE}/python-app-deploy/
        '''
      }
    }

    stage('Run Application') {
      steps {
        sh '''
          . venv/bin/activate
          nohup python ${WORKSPACE}/python-app-deploy/app.py > ${WORKSPACE}/python-app-deploy/app.log 2>&1 &
          echo $! > ${WORKSPACE}/python-app-deploy/app.pid
        '''
      }
    }

    stage('Test Application') {
      steps {
        sh '''
          sleep 2
          python -m unittest discover -s .
        '''
      }
    }
  }
  post {
    success { echo 'Pipeline completed successfully!' }
    failure { echo 'Pipeline failed. Check console.' }
  }
}
