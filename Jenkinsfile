pipeline {
    agent {
        label 'ubuntu-maven'
    }

    tools {
        maven 'mymaven'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Archive WAR') {
            steps {
                archiveArtifacts artifacts: 'target/*.war',
                                 fingerprint: true
            }
        }
        stage('Deploy') {
            steps {
                sshagent(credentials: ['tomcatkey']) {
                    sh '''
                        set -eu
                        WAR=$(find target -maxdepth 1 \
                            -type f -name '*.war' -print -quit)
                        test -n "$WAR"
        
                        scp -o StrictHostKeyChecking=yes \
                            "$WAR" \
                            ec2-user@100.53.233.34:/tmp/swiggy-deploy.war
        
                        ssh -o StrictHostKeyChecking=yes \
                            ec2-user@100.53.233.34 '
                                sudo install -o tomcat -g tomcat -m 0644 \
                                    /tmp/swiggy-deploy.war \
                                    /opt/tomcat/webapps/swiggy.war.new
        
                                sudo mv -f \
                                    /opt/tomcat/webapps/swiggy.war.new \
                                    /opt/tomcat/webapps/swiggy.war
        
                                rm -f /tmp/swiggy-deploy.war
                            '
                    '''
                }
            }
        }
    }
}
