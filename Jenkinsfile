pipeline {
    agent any

    parameters {
        string(name:'repoUrl', defaultValue:'https://github.com/tangjoe/spring-boot.git', description:'代码路径')
        string(name:'dev_server', defaultValue:'IP,Port,Name,Passwd', description:'开发服务器(IP,Port,Name,Passwd)')
        string(name:'prod_server', defaultValue:'IP,Port,Name,Passwd', description:'生产服务器(IP,Port,Name,Passwd)')
    }

    tools {
        maven 'maven3.6'
        jdk   'jdk8'
    }

    post {
        always {
            echo "//Post === clear workspace ==="
            echo "//deleteDir()"
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
        stage('部署到开发环境') {
            steps {
                echo "//Stage-0 === 部署到开发环境 ==="
                script {
                    def dev_split=params.dev_server.split(",")
                    dev_serverIP=dev_split[0]
                    dev_serverPort=dev_split[1]
                    dev_serverName=dev_split[2]
                    dev_serverPasswd=dev_split[3]
                }
                echo "Dev: ${dev_serverName}:${dev_serverPasswd}@${dev_serverIP}:${dev_serverPort}"
            }
        }
        stage('Checkout source') {
            steps {
                echo "//Stage-1 === Checkout source ==="
                git url:params.repoUrl
            }
        }
        stage('Code Qualify Check via SonarQube') {
            steps {
                echo "//Stage-2 === Code Quality Check via SonarQube  ==="
                script {
                    def scannerHome = tool 'SonarQube Scanner';
                        withSonarQubeEnv("SonarQube") {
                            sh "${tool("SonarQube Scanner")}/bin/sonar-scanner \
                                -Dsonar.projectKey=hello-sb \
                                -Dsonar.sources=. \
                                -Dsonar.css.node=. \
                                -Dsonar.host.url=http://sonarqube:9000 \
                                -Dsonar.login=admin \
                                -Dsonar.password=admin"
                        }
                }
            }
        }
        stage('Compile') {
            steps {
                echo "//Stage-3 === complile project ==="
                sh 'mvn clean test'
            }
        }
        stage('Build Fat Jars') {
            steps {
                echo "//Stage-4 === build jars ==="
                sh 'mvn package'
            }   
        }   
        stage('Build docker image') {
            steps {
                echo "//Stage-5 === build docker ==="
                sh 'mkdir -p target/dependency; cd target/dependency; jar -xf ../*.jar'
                sh 'docker build -t hello-sb -f Dockerfile.spring-boot .'
            }
        }
    }
}
