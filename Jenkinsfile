pipeline {
    agent any

    parameters {
        string(name:'repoUrl', defaultValue:'https://github.com/tangjoe/spring-boot.git', description:'代码路径')
        string(name:'dev_server', defaultValue:'IP,Port,Name,Passwd', description:'开发服务器(IP,Port,Name,Passwd)')
        string(name:'prod_server', defaultValue:'IP,Port,Name,Passwd', description:'生产服务器(IP,Port,Name,Passwd)')
    }

    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "nexus:8081/nexus"
        NEXUS_REPOSITORY = "maven-private-repo"
        NEXUS_CREDENTIAL_ID = "maven-login-on-nexus"
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
                                -Dsonar.login=9e90f99c041ef33609b50f60d205c6261e8e7ff5"
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
        stage('Maven to Nexus Repository Manager') {
            steps {
                echo "//Stage-5 === publish to nexus ==="
                script {
                    pom = readMavenPom file: "pom.xml";
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}";
                    artifactPath = filesByGlob[0].path;
                    artifactExists = fileExists artifactPath;
                    if (artifactExists) {
                        echo "*** File:${artifactPath},group:${pom.groupId},packaging:${pom.packaging},version:${pom.version}";

                        nexusArtifactUploader (
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                [artifactId: pom.artifactId,
                                 classifier: '',
                                 file: artifactPath,
                                 type: pom.packaging],
                                [artifactId: pom.artifactId,
                                 classifier: '',
                                 file: "pom.xml",
                                 type: "pom"]
                            ]
                        );
                    } else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
                }
            }
        }
        stage('Build docker image') {
            steps {
                echo "//Stage-6 === build docker image ==="
                sh 'mkdir -p target/dependency; cd target/dependency; jar -xf ../*.jar'
                sh 'docker build -t hello-sb -f Dockerfile.spring-boot .'
            }
        }
        stage('Push docker image to Nexus') {
            steps {
                echo "//Stage-7 === push docker image ==="
                sh 'docker login -u docker -p P@ssw0rd localhost:8082'
                sh 'docker tag hello-sb localhost:8082/hello-sb:1.0'
                sh 'docker push localhost:8082/hello-sb:1.0'
            }
        }
    }
}
