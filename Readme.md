

Current Jenkins Pipeline Script for Deployment

```groovy
pipeline {
    agent any

    stages {
        stage('Copy Frontend') {
            steps {
                git branch: 'main', credentialsId: 'github-jenkins-account', url: 'https://github.com/EZCampusDevs/frontend.git'

                echo 'Imported React Frontend onto VPS'
            }
        }

        stage('Remote Deploy Repository') {
            steps {
                dir('frontend') {
                    sshPublisher(publishers: [
                        sshPublisherDesc(configName: '2GB_Glassfish_VPS', transfers: [
                            sshTransfer(
                                cleanRemote: true, excludes: '', execCommand: '', execTimeout: 120000, flatten: false,
                                makeEmptyDirs: true, noDefaultExcludes: false, patternSeparator: '[, ]+',
                                remoteDirectory: './frontend', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '**/*'
                            )
                        ],
                        usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)
                    ])
                }

                echo 'Repository copied'
            }
        }
		
	stage('Create Docker Container') {
    steps {
        sshPublisher(publishers: [
            sshPublisherDesc(configName: '2GB_Glassfish_VPS', transfers: [
                sshTransfer(
                    cleanRemote: false, excludes: '', execCommand: '''
                        docker run -d --name react-instance \
                            -p 3000:3000 \
                            -v /home/jenkinsslave/frontend/client:/app/frontend/client \
                            alpine sh -c "
                                apk update &&
                                apk add nodejs npm &&
                                cd /app/frontend/client &&
                                npm i -g npm@latest &&
                                npm install &&
                                npm run start
                            "
                    ''',
                    execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false,
                    patternSeparator: '[, ]+', remoteDirectory: './frontend', remoteDirectorySDF: false,
                    removePrefix: '', sourceFiles: ''
                )
            ],
            usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)
        ])
    }
}


        
    }
}
```
