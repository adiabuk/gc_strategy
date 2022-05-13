
pipeline {

    agent any
    environment {
        image_id = "${env.BUILD_ID}"
    }

    stages {
        stage("get config"){
            steps {
                script {
                    sh "pwd; ls"
                    env.WORKSPACE = pwd()
                    def props = readProperties file:'config.ini'
                    def pair= props['pair']
                    def interval= props['interval']
                    echo "Var1=${pair}"
                    echo "Var2=${interval}"
                }
            }
        }
        stage("Clone repo"){
            steps {
                sh 'git clone https://github.com/adiabuk/greencandle.git -b master'
                dir('greencandle') {
                    sh 'git checkout -f $commit'
                    sh 'sudo ln -s `pwd` /srv/greencandle'
                }
            }
        }
        stage("Run tests"){
           steps {
               dir('greencandle') {
                   sh "env"
                    echo "test Var1=${pair}"
                    echo "test Var2=${interval}"
                   sh "docker-compose -f docker-compose_jenkins.yml -p $BUILD_ID up -d unit-runner redis-unit mysql-unit"
                   sh "docker exec unit-runner-$BUILD_ID /docker-entrypoint.sh backend_test -i $interval -d . -p $pair -s"
               }
           }
        }
    }
    post {
        always {
            dir('greencandle'){
                sh "docker-compose -f docker-compose_jenkins.yml -p $BUILD_ID down --rmi all"
                sh "docker network prune -f"
            }
        }
    }
}




