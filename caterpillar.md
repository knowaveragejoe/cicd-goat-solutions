As the `alice` user, have write access to the Caterpillar repository, but are unable to merge pull requests into the main branch.

There are two jenkins pipelines for the project, `wonderland-caterpillar-test` and `wonderland-caterpillar-prod`. Test is triggered by any pull request, main is only triggered by successful merges into main.

Since we can push an updated Jenkinsfile to the repository and trigger the test pipeline, it's fairly easy to dump out the environment present on the Jenkins builder. Doing so reveals a Gitea token which has access to the Gitea API.

We can use the API to merge the pull request into main, and thereby trigger the main pipeline, which will then run our compromised Jenkisfile and print the `flag2` credential using the same `base64` encoding trick as the earlier flags.

```
curl --request POST \
  --url http://localhost:3000/api/v1/repos/Wonderland/caterpillar/pulls/{id}/merge \
  --header 'Authorization: token {token}' \
  --header 'Content-Type: application/json' \
  --header 'accept: application/json' \
  --data '{
  "Do": "merge",
  "force_merge": true
}'
```


