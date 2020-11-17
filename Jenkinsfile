pipeline {
    agent any

    parameters {
        string(name:'repoUrl', defaultValue:'https://github.com/tangjoe/helloworld.git', description:'代码路径')
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
            echo "// clear workspace"
            // deleteDir()
        }
        failure {
            echo "// pipeline job failure"
        }
    }

    stages {
        stage('git: checkout source') {
            steps {
                echo "// git: checkout source"
                git url:params.repoUrl
            }
        }
        stage('sonarqube: code qualify check') {
            steps {
                echo "// sonarqube: code qualify check"
                script {
                    def scannerHome = tool 'SonarQube Scanner';
                        withSonarQubeEnv("SonarQube") {
                            sh "${tool("SonarQube Scanner")}/bin/sonar-scanner \
                                -Dsonar.projectKey=helloworld \
                                -Dsonar.sources=. \
                                -Dsonar.css.node=. \
                                -Dsonar.host.url=http://sonarqube:9000 \
                                -Dsonar.login=9e90f99c041ef33609b50f60d205c6261e8e7ff5"
                        }
                }
            }
        }
        stage('maven: compile & test') {
            steps {
                echo "// maven: compile & test"
                sh 'mvn clean test'
            }
        }
        stage('maven: build fat jars') {
            steps {
                echo "// maven: build fat jars"
                sh 'mvn package'
            }   
        }   
        stage('nexus: push jars to Nexus') {
            steps {
                echo "// nexus: push jars to Nexus"
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
        stage('docker: build image') {
            steps {
                echo "// docker: build image"
                sh 'mkdir -p target/dependency; cd target/dependency; jar -xf ../*.jar'
                sh 'docker build -t helloworld -f Dockerfile.helloworld .'
            }
        }
        stage('clair-scanner: image security scan') {
            steps {
                sh '''
                  echo "// clair-scanner: image security scan"
                  IP=$(ip r | tail -n1 | awk '{ print $9 }')
                  wget -qO clair-scanner https://github.com/arminc/clair-scanner/releases/download/v8/clair-scanner_linux_amd64 && chmod +x clair-scanner
                  ./clair-scanner --ip $IP --clair=http://clair:6060 --threshold="Critical" helloworld:latest || exit 0
                '''
            }
        }
        stage('docker: push image to Nexus') {
            steps {
                echo "// docker: push image to Nexus"
                sh 'docker login -u docker -p P@ssw0rd localhost:8082'
                sh 'docker tag helloworld localhost:8082/helloworld:1.0'
                sh 'docker push localhost:8082/helloworld:1.0'
            }
        }
        stage('docker: startup container') {
            steps {
                script {
                    echo "// docker: startup & test container"
                    docker.image('localhost:8082/helloworld:1.0')
                    .withRun('-p 9080:8080 --network cicd_net --name helloworld --hostname helloworld') {
                        timeout(time: 120, unit: 'SECONDS') {
                            waitUntil {
                                def r = sh script:
                                // Note:
                                // Use container internal port '8080', external port doesn't work
                                //
                                'curl http://helloworld:8080/', returnStatus: true
                                return (r == 0);
                            }
                        }
                    }
                }
            }
        }
    }
}
