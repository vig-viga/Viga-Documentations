# Postman SSO authentication automation
## Pre requisites:
- Postman(Latest version)
- SSO server(status:running)
- MovieColab server(status:running)

### Step 1 - Create Postman environment:
- Create a new environment in postman by clicking the gear icon in the upper right corner of the Postman app the windows "Manage Environments".
- Click the Add button to create a new environment.
- Add the variables as shown in the image below.
![](env.png)
- Check for the variabales while setting them in the environment
-- authService: The full URL to the service handling the authentication,
--  username & password: Your username and password,
-- gatewayBaseUrl: This is the entrypoint of your application(s). 
- Then click on Add to confirm changes.
- Click on Globals and set the variables as shown in the image below.
![](env2.png)

### Step 2: Configure API Collection:
- Create or import an API Collection.
- Right click on the collection and select edit.
- Set the Authentication configuration on this folder as shown in the image and click on Update.
![](env3.png)

### Step 3: Configure Pre-request script:
- Go to the "Edit" of API collection and select "Pre-request Scripts".
- Paste the given script and you are good to go.
```
var authServiceUrl = pm.environment.get('authService');
var gatewayBaseUrl = pm.environment.get('gatewayBaseUrl');
var username = pm.environment.get('email');
var password = pm.environment.get('password');

var sdk = require('postman-collection');

var isValidTokenRequest = new sdk.Request({
    url: gatewayBaseUrl + "/project/", // Use an endpoint that requires being authenticated
    method: 'GET',
    header: [
        new sdk.Header({
            key: 'content-type',
            value: 'application/json',
        }),
        new sdk.Header({
            key: 'acccept',
            value: 'application/json',
        }),
        new sdk.Header({
            key: 'Authorization',
            value: 'Bearer ' + pm.globals.get("jwttoken"),
        }),
    ]
});

pm.sendRequest(isValidTokenRequest, function (err, response) {
    if (response.code === 401) {
        refreshToken();
    }
});

function refreshToken() {
    var tokenRequest = new sdk.Request({
    url: authServiceUrl,
    method: 'POST',
    header: [
        new sdk.Header({
            key: 'content-type',
            value: 'application/json'
        }),
        new sdk.Header({
            key: 'acccept',
            value: 'application/json'
        }),
    ],
    body: {
        mode: 'raw',
        raw: JSON.stringify({
            email: username,
            password: password
        })
    } 
  });

  pm.sendRequest(tokenRequest, function (err, response) {
      if (err) {
          throw err;
      }
      
      if (response.code !== 200) {
          throw new Error('Could not log in.');
      }
      
      pm.globals.set("jwttoken", response.json().access);
      console.log(`New token has been set: ${response.json().access}`);
    //   console.log(`Jwt currcvalue: ${pm.globals.get}`);
  });
}
```

### P.S.:
- Check the console for errors.