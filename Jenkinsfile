pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('DeployToStaging') {
            when {
                branch 'master'
            }
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: '3.83.240.36', keyFileVariable: 'PRIVATEKEY', usernameVariable: 'USERNAME')]) {
                    sshPublisher(
                        failOnError: true,
                        continueOnError: false,
                        sshPublisherDesc(configName: 'staging', sshCredentials: [encryptedPassphrase: '{AQAAABAAAAAQiPLnQ1KNSYBKNwstD3X+u3eMsfe+YrSByO0Eq9ufW0M=}', key: '''-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEAgoWWwoyT1BW94G8QZE7BH+KsUDz5/3lG7OUxYchGSvDzsS4q
vmJrvfs66DG/uUEXPGAQtbAZ12Wx0qvi9bqYVMN2TqqyEwjssz0wBDadqChOsqHh
LrrkdYwmF4s3kEiM7kd6oPZ88pS71wmpok+SZglxzr15JgiBu5r1CnuuBUhh4tMz
JM4Cbji52WnPhfqvStOBi7wEuCqdv3o0eMjo88nS+jp6gBIFYsK1oB1TKb94ova3
nq3xliEZ8NkOtFVn0k9yWOsL8ZHOjy52THHzz7UpzJrRn+hYT1XuiovTkLIaWDLo
nvC960Ks9P5cqfSIRmpsDKpXlocz/DDUPDKyhQIDAQABAoIBAHlVLDfArO+cIn3P
YUKN/3PfqOWSOahnGirASK6omceyxCcyTqPbJGNgd3tAPAU/4BTNDNuJUAxvSeYY
yYw7IL6zUXiBr7aINlnNCKTyDI80oSvn1kg1jolDdmmujkF/YBtlsTaOzMpIv3GA
VwQ+yk42e2h/tG5JvCglPaO4I6ZeNRcfRQuva05uICEWEhfbwyZG/TkBJSGHd022
AmQ8Vl6RgSIVpwFPg3+Kvv8G+NASxiX2eCKmh/LjNjAOGRSYSJG/mHEVpHtk/SFW
7xq+P0Bg3T0WTyAD4tOj++Nnuua8+Vbgl5QVoscVzNTfu7hJp7cicY8dskqSaRLa
NQpimqUCgYEAx1d1MaAgNJR/QfainsntvdJjSyRMKElu+HuWSjESCT7moX2m3yW8
0hT56AEH+5fdi6PQkWZqUwsVIszTLXxjbMQn/rGeV8m96QOPoEYs2hwgyQEynSmV
UutvJaV5/ST/o8oK2vTkF6Y9KPicJUf/rO21jYrBYllRZ+gAtUR0McMCgYEAp56m
F0oGbMNo8DbTf1I2+d/VMYivoFDHSBwYhip07/TK2z74umw8R2xDzXmtVHsuQ1Y4
hjs6XBcvAXoA7Mz0uD9FWS+xXGIt5LhtJ1OejIcRKtHa1/yjRrVhK7QsHI8WTHfr
bLT/vc9/MruQpoJi2Vn3aepr9FN9k+JSRF8nPhcCgYEAwgGtTnIARgwsWl+WX6I5
XqAvAQe/kwn9FZr6ZxSg8Bymy48F5HHO9ktx+Ulfoo5oESqKp6gcXNwRYwAjm0ZP
YaD7J9doOxpeWTSdCSijKFdt1RL0Vp8M9Fmsn+AP/L6QirDtpbHnd9jT91cWaPZM
sNnc83eSxIwXPWA/qCuKRo8CgYA/o5pHqVE7Jg3Hdelio6I/yF23CsAzS7f9hr1A
4wm2uOfzybrBfKp+K3qqnpHSS1pLMocPX5lOsXalRt7nCQG4mj0IGXVrL9NgzSyU
C8lcbUN1UGuYKVEMXXzhDOzagvDiY308rpOSSx9t8Yx/y20gPGoVCm2mO7Sn2vuE
/eN2TQKBgF5IV0uUhaeIJgCkxTBeen4x89+HDnqwxtfsjW7SVrjKIVER+hwjg8an
FJPhwhtpNMoHdOwYiriTSu7Eu30GM36Vgdib9cytM1NtZqoiaSHQB2Xbfmpwo1t9
9wrCHXPHJEO7QPjONVkhH+K2HJDydJ52swVrhXamInklaSPCxZcx
-----END RSA PRIVATE KEY-----%''', keyPath: '', username: 'ec2-user']
                              , usePromotionTimestamp: false, 
                                useWorkspaceInPromotion: false, 
                                verbose: true)], 
                                transfers: [
                                    sshTransfer(
                                        sourceFiles: 'dist/trainSchedule.zip',
                                        removePrefix: 'dist/',
                                        remoteDirectory: '/tmp',
                                        execCommand: 'sudo /usr/bin/systemctl stop train-schedule && rm -rf /opt/train-schedule/* && unzip /tmp/trainSchedule.zip -d /opt/train-schedule && sudo /usr/bin/systemctl start train-schedule'
                                    )
                                ]
                            )
                        ]
                    )
                }
            }
        }
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            steps {
                input 'Does the staging environment look OK?'
                milestone(1)
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    sshPublisher(
                        failOnError: true,
                        continueOnError: false,
                        publishers: [
                            sshPublisherDesc(
                                configName: 'production',
                                sshCredentials: [
                                    username: "$USERNAME",
                                    encryptedPassphrase: "$USERPASS"
                                ], 
                                transfers: [
                                    sshTransfer(
                                        sourceFiles: 'dist/trainSchedule.zip',
                                        removePrefix: 'dist/',
                                        remoteDirectory: '/tmp',
                                        execCommand: 'sudo /usr/bin/systemctl stop train-schedule && rm -rf /opt/train-schedule/* && unzip /tmp/trainSchedule.zip -d /opt/train-schedule && sudo /usr/bin/systemctl start train-schedule'
                                    )
                                ]
                            )
                        ]
                    )
                }
            }
        }
    }
}
