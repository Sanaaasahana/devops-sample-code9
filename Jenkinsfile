pipeline {
  agent any

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Prepare / Build') {
      steps {
        echo 'Checking Python and creating venv...'
        sh '''
          if command -v python3 >/dev/null 2>&1; then
            echo "python3 found: $(python3 --version)"
          else
            echo "python3 NOT found — aborting. Install python3 in the Jenkins node or use Option B." 1>&2
            exit 2
          fi

          python3 -m venv venv || exit 1
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
    failure { echo 'Pipeline failed. Check console logs.' }
  }
}
