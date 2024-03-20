pipeline {
    agent any
    tools {
		jfrog 'jfrog-cli'
	}

    environment {
        SEMGREP_APP_TOKEN = credentials('Semgrep')
        
	
        
    }


    stages {
        stage('Git checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/ipreetgs/microService_TXApplication.git']])
            }
        }
	   
       
	stage('OWSP ZAP VAPT') {
            steps {
		catchError(buildResult: 'Success', stageResult: 'Success'){
                sleep time: 30, unit: 'SECONDS'
                sh 'docker run --rm -v $(pwd):/zap/wrk/:rw -t owasp/zap2docker-stable zap-baseline.py -t http://192.168.6.118:8090 -x xml_report.xml -r zap-report.html'
		sh 'echo Scan Done'
	}
	}
        }

        stage('Publish ZAP Report') {
            steps {
                publishHTML([
                    allowMissing: false,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: '.',
                    reportFiles: 'zap-report.html',
                    reportName: 'ZAP Report',
                    reportTitles: 'ZAP Report'
                ])
            }
        }
	      stages {
        stage('Semgrep-Scan') {
            steps {
                script {
                    // Pull the Semgrep Docker image
                    docker.image('returntocorp/semgrep').pull()
                    
                    // Run Semgrep scan
                    docker.image('returntocorp/semgrep').run(
                        "-e SEMGREP_APP_TOKEN=${env.SEMGREP_APP_TOKEN}",
                        "-v ${pwd()}:${pwd()} --workdir ${pwd()}",
                        'returntocorp/semgrep',
                        'semgrep ci'
                    )
                }
            }
        }
    }
      

}
