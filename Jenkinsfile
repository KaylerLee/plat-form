pipeline {
    agent any
    tools {
        maven "Maven3.6.3"
        jdk "JDK"
    }



    environment {
        // This can be nexus3 or nexus2
        NEXUS_VERSION = "nexus3"
        // This can be http or https
        NEXUS_PROTOCOL = "http"
        // Where your Nexus is running
        NEXUS_URL = "10.97.72.168:8081"
        // Repository where we will upload the artifact
        NEXUS_REPOSITORY = "exo-artifacts"
        // Jenkins credential id to authenticate to Nexus OSS
        NEXUS_CREDENTIAL_ID = "nexusCredential"
        ARTIFACT_VERSION = "${BUILD_NUMBER}"
    }

    stages {

        stage("Check out") {
            steps {
                script {
                    git branch: 'master', url: 'https://github.com/KaylerLee/plat-form.git';
                }
            }
        }

        stage("mvn clean install -DskipTests=true") {
            steps {
                script {
                    sh "mvn clean package -DskipTests=true"
                }   
            }
        }

         stage("publish to nexus") {
            steps {
                script {
                    // Read POM xml file using 'readMavenPom' step, this step 'readMavenPom' is included in: https://plugins.jenkins.io/pipeline-utility-steps
                    pom = readMavenPom file: "pom.xml";
                    // Find built artifact under target folder
                   filesByGlob = findFiles(glob: "*/target/*.jar");
                    echo "${filesByGlob.size()}";
                    // Print some info from the artifact found

                    for (int i = 0; i < filesByGlob.size(); i++) {
                     
                    echo "${filesByGlob[i].name} ${filesByGlob[i].path} ${filesByGlob[i].directory} ${filesByGlob[i].length} ${filesByGlob[i].lastModified}";
                    // Extract the path from the File found
                    artifactPath = filesByGlob[3].path;
                    // Assign to a boolean response verifying If the artifact name exists
                    artifactExists = fileExists artifactPath;
                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";
                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: ARTIFACT_VERSION,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                // Artifact generated such as .jar, .ear and .war files.
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging]
                            ]
                        );

                    } else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
                }
                }
                }
            }
        }
    }