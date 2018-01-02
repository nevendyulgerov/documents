# Fantasy POS - Build Procedures

An overview of the build procedures available in the Fantasy POS app.

Fantasy POS utilizes [gulp](https://gulpjs.com/) and [npm](https://www.npmjs.com/) for its build tools. The app is integrated with a number of procedures to facilitate daily development tasks such as:

- running a local development server
- running an automated build, allowing for preprocessing and concatenation of sass/js resources
- creating a distributable package with the latest app's revision

## Running a local development server

Fantasy POS comes packed with [http-server](https://www.npmjs.com/package/http-server). It provides the web server for local development. To start the server run: `npm run server`

The app will become available at `http://localhost:8080`

## Running an automated build

The app also comes with sass/js build tools. The primary build task enables sass preprocessing and js concatenation of all meaningful resources upon change. To start this ongoing task run: `npm run build`

## Creating a distributable package

You can easily create a distributable package using the latest app code. To run this use: `npm run dist`. Note, that the distributable will be located under `appDir/dist/`

## Extending the build tools

All build tasks reside in appDir/gulpfile.js. Make sure to add/modify tasks from there.
