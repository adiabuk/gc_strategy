
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
                    env.pairs= props['pairs']
                    env.intervals= props['intervals']
                    env.name = props['name']
                    env.year = props['year']
                    echo "Var1=${pairs}"
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
                   env
                   export id=${BUILD_ID}-${JOB_BASE_NAME}
                   export test=strat-${BUILD_ID}-${JOB_BASE_NAME}
                   docker-compose -f install/docker-compose_jenkins.yml -p ${BUILD_ID}-${JOB_BASE_NAME} up -d unit-runner redis-unit mysql-unit
                   docker cp ../greencandle.ini unit-runner-${BUILD_ID}-${JOB_BASE_NAME}:/etc/greencandle.ini
                   docker exec unit-runner-${BUILD_ID}-${JOB_BASE_NAME} bash -c 'mkdir -p /data/output/${name} ; chmod 777 /data/output/${name}'
                   sleep 60
                   """
                   script {
                       def arr = env.intervals.split(",")
                       def arr2 = env.pairs.split(",")
                       for (interval in arr) {
                          for (pair in arr2) {
                             println "Running interval ${interval}"
                             sh """
                             export id=${BUILD_ID}-${JOB_BASE_NAME}
                             export test=strat-${BUILD_ID}-${JOB_BASE_NAME}
                             docker exec unit-runner-${BUILD_ID}-${JOB_BASE_NAME} bash -c 'backend_test -i $interval -d /data/altcoin_historical/${year}/sept -p ${pair} -s 2>&1 | tee /data/output/${name}/${pair}-${interval}-${year}.log'
                             docker exec unit-runner-${BUILD_ID}-${JOB_BASE_NAME} bash -c 'report ${interval} /data/output/${name}/${pair}-${interval}-${year}.xlsx'
                             docker exec unit-runner-${BUILD_ID}-${JOB_BASE_NAME} bash -c 'create_graph -p ${pair} -i ${interval} -o /data/output/${name}'
                             docker exec unit-runner-${BUILD_ID}-${JOB_BASE_NAME} bash -c 'cp /etc/greencandle.ini /data/output/${name}/greencandle.ini.${pair}-${interval}'
                             """
                           }
                       }
                   }
               }
           }
        }
    }
    post {
        always {
            dir('greencandle'){
                sh "docker-compose -f docker-compose_jenkins.yml -p ${BUILD_ID}-${JOB_BASE_NAME} down --rmi all"
                sh "docker network prune -f"
            }
        }
    }
}




