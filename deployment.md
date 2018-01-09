# Fantasy POS - Deployment

The deployment procedure currently implemented in Fantasy POS is somewhat complex. There are several steps you need to take in order to deploy a new revision of the app to the staging/production environments:

- Create a distributable package using `npm run dist`
- Copy the contents of appDir/dist
- Access the remote server
- On the remote server go to C:/WebSites/fieldbank/dist and paste the contents of appDir/dist
- Go back to C:/WebSites/fieldbank and run a terminal from this path
- Execute `node deploy-fantasy {environment} {user}`, where {environment} can be stage/prod and {user} is a user defined in C:/WebSites/fieldbank/config/config.js - deploy - users
- Confirm the operation when prompted

`Note:` If you have successfully deployed the new code and are added to the slack channel 'fantasy-deployments' you should be able to see there a message for the new deploy.

`Note:` After you have successfully deployed the new code, make sure to commit the new revision (available at appDir/app.json) to prevent any inconsistencies in the revision number.

`Note:` Automation solutions for the above process were considered. However, due to security restrictions (the repo is available via VPN, the server is accessible via remote access) these solutions were not sufficient. A developer experienced in shell scripting may find a better alternative to the current deployment procedure.
