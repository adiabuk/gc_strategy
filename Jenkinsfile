
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
                    env.intervals= props['intervals'].split(',')
                    env.name = props['name']
                    env.year = props['year']
                    echo "Var1=${pair}"
                    echo "Var2=${intervals}"
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
                   env.intervals.each() {
                       echo it
                   }
                   sh "sleep 1000"
                   sh "env"
                   sh "docker-compose -f docker-compose_jenkins.yml -p $BUILD_ID up -d unit-runner redis-unit mysql-unit"
                   sh "docker cp ../greencandle.ini unit-runner-${BUILD_ID}:/etc/greencandle.ini"
                   sh "docker exec unit-runner-$BUILD_ID bash -c 'mkdir -p /data/output/${name} ; chmod 777 /data/output/${name}'"
                   sh "sleep 60"
                   sh "docker exec unit-runner-$BUILD_ID bash -c 'backend_test -i $interval -d /data/altcoin_historical/${year}/year -p $pair -s 2>&1 | tee /data/output/${name}/${pair}-${interval}-${year}.log'"
                   sh "docker exec unit-runner-$BUILD_ID bash -c 'report ${interval} /data/output/${name}/${pair}-${interval}-${year}.xlsx'"
                   sh "docker exec unit-runner-$BUILD_ID bash -c 'create_graph -p ${pair} -i ${interval} -o /data/output/${name}'"

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




