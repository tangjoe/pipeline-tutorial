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
        stage('Deploy') {
            steps {
                echo "//Stage-4 === deploy jars ==="
                sh 'java -jar target/myproject-0.0.1-SNAPSHOT.jar --server.port=8989 &'
            }
        }
        stage('Run') {
            steps {
                echo "//Stage-5 === run ==="
                script {
                    final String url = "http://localhost:8989"
                    final String response = sh(script: "curl -s $url", returnStdout: true).trim()
                    echo response
                }
            }
        }
    }
}
