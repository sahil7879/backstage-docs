# authentication setup using github 
official documentation page : https://backstage.io/docs/auth/#sign-in-configuration
## For this you have to create github oauth app first 
### Step 1  
- go to your github homepage  
- click on the profile icon at the top right corner  
- scroll down to the end in developer options  
- select oauth app  
- configure it similar to this  
![ github oauth app Screenshot](/images/screenshot1.png)
- copy client id and client secret and save them at a safe place

### Step 2
edit app.config.local.yaml file and add this block 
```
auth:
  environment: development
  providers:
    github:
      development:
        clientId: ${AUTH_GITHUB_CLIENT_ID}
        clientSecret: ${AUTH_GITHUB_CLIENT_SECRET}
        ## uncomment if using GitHub Enterprise
        # enterpriseInstanceUrl: ${AUTH_GITHUB_ENTERPRISE_INSTANCE_URL}
        ## uncomment to set lifespan of user session
        # sessionDuration: { hours: 24 } # supports `ms` library format (e.g. '24h', '2 days'), ISO duration, "human duration" as used in code
        signIn:
          resolvers:
            # See https://backstage.io/docs/auth/github/provider#resolvers for more resolvers
            - resolver: usernameMatchingUserEntityName
```

### Step 3  
Now add frontend part of the authentication   
edit this file packages/app/src/App.tsx and add  
```
import { githubAuthApiRef } from '@backstage/core-plugin-api';
import { SignInPage } from '@backstage/core-components';

const app = createApp({
  components: {
    SignInPage: props => (
      <SignInPage
        {...props}
        auto
        provider={{
          id: 'github-auth-provider',
          title: 'GitHub',
          message: 'Sign in using GitHub',
          apiRef: githubAuthApiRef,
        }}
      />
    ),
  },
  // ..
});
```
also check you are not duplicating anything
### Step 4  
now lets add the backend too 
```
yarn --cwd packages/backend add @backstage/plugin-auth-backend-module-github-provider
```
and add this line in the file at this location packages/backend/src/index.ts
```
backend.add(import('@backstage/plugin-auth-backend-module-github-provider'));
```

### Step 5
Now lets add a user in catalog so that we can authenticate
add a folder catalog/entities in the root directory of the app
and add a file name users in entities folder
enter these details in the file 
```
---
apiVersion: backstage.io/v1alpha1
kind: User
metadata:
  name: YOUR GITHUB USERNAME
spec:
  memberOf: [guests]
```
now add the location of the file in app.config.local.yaml file 
```
catalog:
  rules:
    rules:
    - allow: [User, Group, Component, API, Location, Template]
  locations:
    - type: file
      target: /home/ubuntu/backstage-app/backstage/catalog/entities/Users.yaml
```
### Step 6
now to check whether the authentication work start the app
```
yarn start
```
at the homepage you will see auth with github  
enter username and password and you will be able to log in  
now check the setting there you will see your profile logo and username  
