
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
                    env.pair= props['pair']
                    env.interval= props['interval']
                    env.name = props['name']
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
                   sh "mkdir /data/output/${name}"
                   sh "docker-compose -f docker-compose_jenkins.yml -p $BUILD_ID up -d unit-runner redis-unit mysql-unit"
                   sh "docker cp greencandle.ini unit-runner-${BUILD_ID}:/etc/greencandle"
                   sh "docker exec unit-runner-$BUILD_ID /docker-entrypoint.sh backend_test -i $interval -d /data/altcoin_historical/year${year} -p $pair -s 2>&1 | tee /data/output/${name}"
                   sh "report ${interval} /data/output/${name}/${pair}-${interval}-${year}.xlsx"
                   sh "create_graph -p ${pair} -i ${interval} -o /data/output/${name}"

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




