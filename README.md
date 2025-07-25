# Library ![Supported node versions](https://img.shields.io/badge/dynamic/json?color=informational&label=node&query=%24.engines.node&url=https%3A%2F%2Fraw.githubusercontent.com%2Fnytimes%2Flibrary%2Fmain%2Fpackage.json) [![Tests](https://github.com/nytimes/library/actions/workflows/test.yaml/badge.svg)](https://github.com/nytimes/library/actions/workflows/test.yaml)

A collaborative newsroom documentation site, powered by Google Docs.

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [Forked from NYT Library](#forked-from-nyt-library)
- [Development Workflow](#development-workflow)
  - [Tests](#tests)
- [Customization](#customization)
- [Deploying the app](#deploying-the-app)
- [App structure](#app-structure)
  - [Server](#server)
  - [Views](#views)
  - [Doc parsing](#doc-parsing)
  - [Listing the drive](#listing-the-drive)
  - [Auth](#auth)
  - [User Authentication](#user-authentication)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Forked from NYT Library

The MVP Library is a fork of this package: [https://github.com/nytimes/library](https://github.com/nytimes/library). Read the Readme there for more details.

## Development Workflow

1. Clone and `cd` into the repo:

   `git clone git@github.com:nytimes/library.git && cd library`

2. From the Google API console, create or select a project, then create a service account with the Cloud Datastore User role. It should have API access to Drive and Cloud Datastore. Store these credentials in `server/.auth.json`.

   - To use oAuth, you will also need to create oAuth credentials.
   - To use the Cloud Datastore API for reading history, you will need to add in your `GCP_PROJECT_ID`.

3. Install dependencies:

   `npm install --no-optional`

4. Create a `.env` file at the project root. An example `.env` might look like

```bash
# node environment (development or production)
NODE_ENV=development
# Google oAuth credentials
GOOGLE_CLIENT_ID=123456-abcdefg.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=abcxyz12345
GCP_PROJECT_ID=library-demo-1234
# comma separated list of approved access domains or email addresses (regex is supported).
APPROVED_DOMAINS="nytimes.com,dailypennsylvanian.com,(.*)?ar.org,demo.user@demo.site.edu"
SESSION_SECRET=supersecretvalue

# Google drive Configuration
# team or folder ("folder" if using a folder instead of a team drive)
DRIVE_TYPE=team
# the ID of your team's drive or shared folder. The string of random numbers and letters at the end of your team drive or folder url.
DRIVE_ID=0123456ABCDEF
```

Make sure to not put any comments in the same line as `DRIVE_TYPE` and `DRIVE_ID` vars.

Ensure you share your base drive or folder with the email address associated with the service account created in step 2.

**Be careful!** Setting NODE_ENV to `development` changes the built in behaviors for site authentication to allow accounts other than those in the APPROVED_DOMAINS list. **Never use NODE_ENV=development for your deployed site, only locally.**

5. Start the app:

   `npm run watch`

The app should now be running at `localhost:3000`. Note that Library requires Node v8 or higher.

### Tests

You can run functional and unit tests, which test HTML parsing and routing logic, with `npm test`. A coverage report can be generated by running `npm run test:cover`.

The HTML parsing tests are based on the [Supported Formats doc](https://docs.google.com/document/d/12bUE9b4_aF_IfhxCLdHApN8KKXwa4gQUCGkHzt1FyRI/edit). To download a fresh copy of the HTML after making edits, run `node test/utils/updateSupportedFormats.js`.

## Customization

Styles, text, caching logic, and middleware can be customized to
match the branding of your organization. This is covered in the [customization readme](https://github.com/nytimes/library/blob/master/custom/README.md).

A sample customization repo is provided at [nytimes/library-customization-example](https://github.com/nytimes/library-customization-example).

## Deploying the app

MVP uses Google App Engine to deploy Library. To deploy changes after they have been committed, run:

```sh
gcloud app deploy
```

Make sure you are in the project root (where `app.yaml` is located) and have the Google Cloud SDK (`gcloud`) installed and authenticated. If you need to specify a project, add `--project=YOUR_PROJECT_ID` to the deploy command.

## App structure

### Server

The main entry point to the app is `index.js`.

This file contains the express server which will respond to requests for docs
in the configured team drive or shared folder. Additionally, it contains logic
about issuing 404s and selecting the template to use based on the path.

### Views

Views (layouts) are located in the `layouts` folder. They use the `.ejs`
extension, which uses a syntax similar to underscore templates.

Base styles for the views are in the `styles` directory containing Sass files.
These files are compiled to CSS and placed in `public/css`.

### Doc parsing

Doc HTML fetch and parsing is handled by `docs.js`. `fetchDoc` takes the ID of a
Google doc and a callback, then passes the HTML of the document into the
callback once it has been downloaded and processed.

### Listing the drive

Traversing the contents of the NYT Docs folder is handled by `list.js`. There
are two exported functions:

- `getTree` is an async call that returns a nested hash (tree) of Google Drive
  Folder IDs mapped to their children. It is used by the server to determine
  whether a route is valid or not.

- `getMeta` synchronously returns a hash of Google Doc IDs to metadata objects
  that were saved in the course of populating the tree. This metadata includes
  edit history, document authors, and parent folders.

The tree and file metadata are repopulated into memory on an interval (currently 60s). Calling getTree multiple times will not return fresher data.

### Auth

Authentication with the Google Drive v3 api is handled by the auth.js file, which exposes a single method `getAuth`. `getAuth` will either return an already instantiated authentication client or produce a fresh one. Calling `getAuth` multiple times will not produce a new authentication client if the credentials have expired; we should build this into the auth.js file later to automatically refresh the credentials on some sort of interval to prevent them from expiring.

### User Authentication

Library currently supports both Slack and Google Oauth methods. As Library sites are usually intended to be internal to a set of limited users, Oauth with your organization is strongly encouraged. To use Slack Oauth, specify your Oauth strategy in your `.env` file, like so:

```
# Slack needs to be capitalized as per the Passport.js slack oauth docs http://www.passportjs.org/packages/passport-slack-oauth2/
OAUTH_STRATEGY=Slack
```

You will need to provide Slack credentials, which you can do by creating a Library Oauth app in your Slack workspace. After creating the app, save the app's `CLIENT_ID` and `CLIENT_SECRET` in your `.env` file:

```
SLACK_CLIENT_ID=1234567890abcdefg
SLACK_CLIENT_SECRET=09876544321qwerty
```

You will need to add a callback URL to your Slack app to allow Slack to redirect back to your Library instance after the user is authenticated.
