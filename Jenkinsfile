pipeline {
    agent any

    environment {
        PROJECT_NAME    = "pipeline-test"
        SONARQUBE_URL   = "http://localhost:9000"
        SONARQUBE_TOKEN = "sqa_e51b9943d0a0049daf3083c8f69becd5f9e3acc3"
        TARGET_URL      = "http://172.26.245.185:5000"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/fernaaandaaaa/proyecto-devsecops.git'
            }
        }

        stage('Create Virtualenv & Install Dependencies') {
            steps {
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }

        stage('SonarQube Analysis') {
    steps {
        script {
            // Nombre EXACTO del Scanner configurado en Jenkins
            def scannerHome = tool 'SonarQubeScanner'

            // Nombre EXACTO del servidor SonarQube definido en "SonarQube servers"
            withSonarQubeEnv('SonarQubeScanner') {

                sh """
                    . venv/bin/activate
                    "${scannerHome}/bin/sonar-scanner" \
                      -Dsonar.projectKey=proyecto-devsecops \
                      -Dsonar.sources=. \
                      -Dsonar.host.url=http://sonarqube:9000
                """
            }
        }
    }
}



       stage('Dependency Check') {
    steps {
        script {
            // Nombre EXACTO de la herramienta configurada en Jenkins > Tools > Dependency-Check
            def dcHome = tool 'DependencyCheck'

            sh """
                ${dcHome}/bin/dependency-check.sh \
                  --scan . \
                  --out dependency-check-report \
                  --format HTML
            """
        }
    }
}

        stage('Run Python App') {
            steps {
                sh '''
                    . venv/bin/activate
                    python3 vulnerable_server.py &
                    sleep 5
                '''
            }
        }
    }

    post {
        always {
            echo "Pipeline finished."
        }
    }
}
