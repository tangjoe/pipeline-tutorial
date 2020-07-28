pipeline {
    agent any

    parameters {
        string(name:'repoUrl', defaultValue:'https://github.com/tangjoe/spring-boot.git', description:'代码路径')
    }

    tools {
        maven 'maven3.6'
        jdk   'jdk8'
    }

    post {
        always {
            echo "//Post === clear workspace ==="
            deleteDir()
            sh 'docker rm -f hello-sb'
        }
        failure {
            echo "//Post === pipeline job failure ==="
        }
    }

    stages {
        stage('清理本地仓库') {
            steps {
                echo "//Stage-0 === 清理本地仓库==="
            }
        }
        stage('Checkout source') {
            steps {
                echo "//Stage-1 === Checkout source ==="
                git url:params.repoUrl
            }
        }
        stage('Compile') {
            steps {
                echo "//Stage-2 === complile project ==="
                sh 'mvn clean test'
            }
        }
        stage('Build Fat Jars') {
            steps {
                echo "//Stage-3 === build jars ==="
                sh 'mvn package'
            }   
        }   
        stage('Build docker image') {
            steps {
                echo "//Stage-4 === build docker ==="
                sh 'mkdir -p target/dependency; cd target/dependency; jar -xf ../*.jar'
                sh 'docker build -t hello-sb -f Dockerfile.spring-boot .'
            }
        }
        stage('Run docker') {
            steps {
                echo "//Stage-5 === run docker ==="
                sh 'docker run -d --name hello-sb -p 8089:8080 hello-sb'
            }
        }
        stage('Test with curl') {
            steps {
                echo "//Stage-6 === test with curl ==="      
                sleep 30
                script {
                    final String url = "http://localhost:8089"
                    final String response = sh(script: "curl -s $url", returnStdout:true).trim()
                    echo response
                }
            }
        }
    }
}
