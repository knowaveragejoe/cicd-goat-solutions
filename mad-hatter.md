# Mad Hatter

In this repository, our user `thealice` only has write access to the main `mad-hatter` repository.
However, the Jenkinsfile is located in `mad-hatter-pipeline`, which we cannot modify.
Fortunately, we can manipulate the `Makefile` in the `mad-hatter` repository and abuse this to dump credentials.
Conveniently, the call to the Makefile is already wrapped in a `withCredentials()` call already.

Altering the Makefile and triggering the pipeline was simple:

```Makefile
whoami:
	echo "${FLAG}" | base64
```
