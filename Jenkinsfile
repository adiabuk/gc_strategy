
pipeline {

    agent any

    stages {
        stage("get config"){
            steps {
                script {
                    sh "pwd; ls"
                    env.WORKSPACE = pwd()
                    def props = read Properties  file:'config.ini'
                    def pair= props['pair']
                    def interval= props['interval']
                    echo "Var1=${pair}"
                    echo "Var2=${interval}"
                }
            }
        }
        stage("Clone repo"){
            steps {
                sh 'git clone https://github.com/adiabuk/greencandle.git -b $version'
                dir('greencandle') {
                    sh 'git checkout -f $commit'
                    sh 'sudo ln -s `pwd` /srv/greencandle'
                }
            }
        }
        stage("Run tests"){
           steps {
               dir('greencandle') {
                   sh "docker-compose -f docker-compose_jenkins.yml -p $BUILD_ID up -d unit-runner redis-unit mysql-unit"
                   sh "docker exec unit-runner-$BUILD_ID /docker-entrypoint.sh backend_test -i $interval -d . -p $pair -s"
               }
           }
        }
    }
}




