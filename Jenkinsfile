pipeline {
    agent any

    environment {
        // Nombre que verá SonarQube como clave de proyecto
        PROJECT_NAME = "proyecto-devsecops"

        // URL de tu app vulnerable (por si la ocupas en otras etapas)
        TARGET_URL   = "http://172.26.245.185:5000"
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

        stage('Python Security Audit (pip-audit)') {
            steps {
                sh '''
                    . venv/bin/activate
                    pip install pip-audit
                    mkdir -p security-reports
                    # Genera reporte en Markdown (no rompe el build si encuentra vulnerabilidades)
                    pip-audit -r requirements.txt -f markdown -o security-reports/pip-audit.md || true
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    // Nombre EXACTO del scanner configurado en Manage Jenkins > Tools
                    def scannerHome = tool 'SonarQubeScanner'

                    // Nombre EXACTO del servidor SonarQube configurado en Manage Jenkins > Configure System
                    withSonarQubeEnv('SonarQubeScanner') {
                        sh """
                            . venv/bin/activate
                            "${scannerHome}/bin/sonar-scanner" \
                              -Dsonar.projectKey=${PROJECT_NAME} \
                              -Dsonar.sources=.
                        """
                    }
                }
            }
        }

        stage('Dependency Check (OWASP)') {
            steps {
                // Usa tu credencial global con ID = nvdApiKey
                withCredentials([string(credentialsId: 'nvdApiKey', variable: 'NVD_API_KEY')]) {
                    script {
                        // Nombre EXACTO de la herramienta en Manage Jenkins > Tools > Dependency-Check
                        def dcHome = tool 'DependencyCheck'

                        // No queremos que el fallo del NVD bote todo el pipeline
                        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                            sh """
                                "${dcHome}/bin/dependency-check.sh" \
                                  --scan . \
                                  --out dependency-check-report \
                                  --format HTML \
                                  --nvdApiKey ${NVD_API_KEY}
                            """
                        }
                    }
                }
            }
        }

        stage('Run Python App') {
            steps {
                sh '''
                    . venv/bin/activate
                    # Levanta el servidor vulnerable en segundo plano
                    python3 vulnerable_server.py &
                    sleep 5
                '''
            }
        }

        stage('Publish Reports') {
            steps {
                // Publica el reporte HTML de OWASP Dependency-Check
                publishHTML([
                    allowMissing: true,                 // por si algún build no genera el HTML
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'dependency-check-report',
                    reportFiles: 'dependency-check-report.html',
                    reportName: 'OWASP Dependency-Check Report'
                ])

                // También puedes guardar el pip-audit.md como artefacto
                archiveArtifacts artifacts: 'security-reports/pip-audit.md', onlyIfSuccessful: false
            }
        }
    }

    post {
        always {
            echo "Pipeline finished."
        }
    }
}
