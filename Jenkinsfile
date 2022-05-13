
pipeline {

    agent any


    environment {
        script {
            readFile "<your config file>"
            configData = file.split("\n")
            configData.each {
                lineData = it.split("=")
                switch(lineData[0].toLowerCase().trim()){
                    case "pair": pair = lineData[1].trim(); break;
                    case "years": years = lineData[1].trim(); break;
                    case "version": version = lineData[1].trim(); break;
                }
            }
        }
    }

    stages {
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




