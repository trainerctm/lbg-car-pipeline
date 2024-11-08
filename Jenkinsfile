pipeline {
    agent any
    environment {
        SERVER_URL = "http://34.76.119.102:8000/" 
        MYSQL_ROOT_PASSWORD = credentials('MYSQL_ROOT_PASSWORD')
    }
    stages {
        stage('Checkout source repos') {
            steps {
                dir("lbg-car-front") {
                    git url: "https://github.com/trainerctm/lbg-car-react-starter.git", branch: "main"
                }
                dir("lbg-car-back") {
                    git url: "https://github.com/trainerctm/lbg-car-spring-app-starter.git", branch: "main"
                }
            }
        }
        stage('Test and build spring backend') {
            steps {
                dir("lbg-car-back") {
                    sh "mvn clean test"
                    sh '''
                    cat - > src/main/resources/application.properties <<EOF
                    spring.profiles.active=prod
                    logging.level.root=DEBUG
                    server.port=8000
                    spring.jpa.show-sql=true
                    '''
                    sh "docker build -t ctm2000/lbg-car-back:v${BUILD_NUMBER} --build-arg MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD} ."
                    sh "docker tag ctm2000/lbg-car-back:v${BUILD_NUMBER} ctm2000/lbg-car-back:latest"
                }
            }
        }
        stage('Test and build react frontend') {
            steps {
                dir("lbg-car-front") {
                    sh """
                    npm install
                    yarn test
                    docker build --build-arg SERVER_URL=${SERVER_URL} -t ctm2000/lbg-car-front:v${BUILD_NUMBER} .
                    docker tag ctm2000/lbg-car-front:v${BUILD_NUMBER} ctm2000/lbg-car-front:latest
                    """
                }
            }
        }
        stage('Push docker images and cleanup') {
            steps {
                sh "docker push ctm2000/lbg-car-front:v${BUILD_NUMBER}"
                sh "docker push ctm2000/lbg-car-front:latest"
                sh "docker push ctm2000/lbg-car-back:v${BUILD_NUMBER}"
                sh "docker push ctm2000/lbg-car-back:latest"
                sh "docker rmi ctm2000/lbg-car-back victorialloyd/lbg-car-back:v${BUILD_NUMBER} ctm2000/lbg-car-front ctm2000/lbg-car-front:v${BUILD_NUMBER}"
            }
        }
        stage('Deploy to server') {
            steps {
                sh "scp -i ~/.ssh/server_key docker-compose.yaml jenkins@34.76.119.102:/home/jenkins/docker-compose.yaml"
                sh """
                ssh -i ~/.ssh/server_key jenkins@34.76.119.102 <<EOF
                export MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
                docker-compose up -d
                """
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
