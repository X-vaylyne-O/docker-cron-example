def getEnvName(branchName) {
	if ("dev".equals(branchName)){
		return "dev";
	} else if("staging".equals(branchName)) {
		return "staging";
	} else if ("master".equals(branchName)) {
		return "master";
	} else {
		return "anyOtherBranch"
	}
}


def getName(branchName) {
	if("dev".equals(branchName)) {
		return "dev_py_cron";
	} else if("staging".equals(branchName)) {
		return "staging_py_cron";
	} else if ("master".equals(branchName)) {
		return "py_cron";
	}
}

def getPortNumber(branchName) {
	if("dev".equals(branchName)) {
		return 9666;
	} else if("staging".equals(branchName)) {
		return 7666;
	} else if ("master".equals(branchName)) {
		return 5666;
	}
}

pipeline {
	triggers {
		pollSCM('H/5 * * * *')
	}
	agent {
		label 'pacific'
	}
	options {
		timeout(time: 1, unit: 'HOURS')
		buildDiscarder(logRotator(daysToKeepStr: '0', numToKeepStr: '0'))
	}
	environment {
		NODE_ENV = getEnvName(env.BRANCH_NAME)
		NAME = getName(env.BRANCH_NAME)
		PORT = getPortNumber(env.BRANCH_NAME)
		HOME = '.'
	}
	stages {
		stage('Build and Test Prod') {
			when {
				expression {
					return NODE_ENV == 'master' ;
				}
			}
			agent {
				dockerfile {
					filename 'Dockerfile'
					additionalBuildArgs '-t ${NAME}'
					label 'pacific'
				}
			}
            steps {
				sh 'apt-get update'
			}
		}
		stage('Prep') {
			when {
				expression {
					return NODE_ENV == 'master';
				}
			}
			steps {
				script {
					try {
						sh 'docker stop ${NAME}'
						sh 'docker rm ${NAME}'
					} catch(Exception e) {
						echo 'Exception occurred: ' + e.toString()
					}
				}
			}
		}
		stage('Deploy master') {
			when {
				expression {
					return NODE_ENV == 'master';
				}
			}
			steps {
				sh '''
					docker run \
						-d \
						-p ${PORT}:5000 \
						--name ${NAME} \
						-e NODE_ENV=${NODE_ENV} \
						-e NAME=${NAME} \
						${NAME}
				'''
			}
		}
		stage('Cleanup') {
			when {
				expression {
					return NODE_ENV == 'master';
				}
			}
			steps {
				sh 'docker image prune -f'
			}
		}
	}
    post {
        always {
            echo 'The pipeline completed'
        }
        success {
            sh "sudo nohup python3 app.py > log.txt 2>&1 &"
            echo "Cron job finally stopped exploding"
        }
        failure {
            echo 'Build stage failed'
            error('Stopping early…')
        }
    }
}
