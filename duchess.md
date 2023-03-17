This is a relatively simple exercise. We're looking for a PyPi token, which are prefixed with `pypi`.
We can simply clone the repo and look for any matches in the commit history like so:

```sh
git clone http://localhost:3000/Wonderland/duchess.git
cd duchess
git log -p | grep pypi -C5
```
