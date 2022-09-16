
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
                    env.intervals= props['intervals']
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
                   sh """
                   export id=${BUILD_ID}
                   export test=strat-${BUILD_ID}
                   docker-compose -f install/docker-compose_jenkins.yml -p $BUILD_ID up -d unit-runner redis-unit mysql-unit
                   docker cp ../greencandle.ini unit-runner-${BUILD_ID}:/etc/greencandle.ini
                   docker exec unit-runner-$BUILD_ID bash -c 'mkdir -p /data/output/${name} ; chmod 777 /data/output/${name}'
                   sleep 60
                   """
                   script {
                       def arr = env.intervals.split(",")
                       for (interval in arr) {
                           println "Running interval ${interval}"
                           sh """
                           export id=${BUILD_ID}
                           export test=strat-${BUILD_ID}
                           docker exec unit-runner-$BUILD_ID bash -c 'backend_test -i $interval -d /data/altcoin_historical/${year}/year -p $pair -s 2>&1 | tee /data/output/${name}/${pair}-${interval}-${year}.log'
                           docker exec unit-runner-$BUILD_ID bash -c 'report ${interval} /data/output/${name}/${pair}-${interval}-${year}.xlsx'
                           docker exec unit-runner-$BUILD_ID bash -c 'create_graph -p ${pair} -i ${interval} -o /data/output/${name}'
                           docker exec unit-runner-$BUILD_ID bash -c 'cp /etc/greencandle.ini /data/output/${name}/greencandle.ini.${pair}-${interval}'
                           """
                       }
                   }
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




