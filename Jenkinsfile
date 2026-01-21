pipeline {
    agent { label 'Node-2' }

    options {
        timestamps()
    }

    environment {
        GIT_REPO = 'git@github.com:Dhananjayakl/c_project.git'
        BRANCH = 'main'
    }

    stages {

        stage('Checkout repo') {
            steps {
                git branch: BRANCH,
                    url: GIT_REPO,
                    credentialsId: '64037187-a6f8-40a4-91a5-4906606e21bf'
            }
        }

        stage('Prepare tools') {
            steps {
                sh '''
                    set -e

                    sudo apt-get update -y
                    sudo apt-get install -y \
                        python3 \
                        python3-pip \
                        cmake \
                        curl \
                        pipx

                    export PATH=$PATH:$HOME/.local/bin

                    pipx install cmakelint || pipx reinstall cmakelint
                    cmakelint --version
                '''
            }
        }

        stage('Lint') {
            steps {
                sh '''
                    export PATH=$PATH:$HOME/.local/bin

                    cmakelint CMakeLists.txt > lint_report.txt 2>&1 || true
                    cat lint_report.txt
                '''
            }
            post {
                always {
                    archiveArtifacts artifacts: 'lint_report.txt', fingerprint: true
                }
            }
        }

        stage('Build') {
            steps {
                sh '''
                    if [ -f CMakeLists.txt ]; then
                        mkdir -p build
                        cd build
                        cmake ..
                        make -j$(nproc)
                    else
                        echo "CMakeLists.txt not found!"
                        exit 1
                    fi
                '''
            }
        }
    }
}
