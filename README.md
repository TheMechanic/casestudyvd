# Candidate Takehome Exercise
This is a simple backend engineer take-home test to help assess candidate skills and practices.  We appreciate your interest in Voodoo and have created this exercise as a tool to learn more about how you practice your craft in a realistic environment.  This is a test of your coding ability, but more importantly it is also a test of your overall practices.

If you are a seasoned Node.js developer, the coding portion of this exercise should take no more than 1-2 hours to complete.  Depending on your level of familiarity with Node.js, Express, and Sequelize, it may not be possible to finish in 2 hours, but you should not spend more than 2 hours.  

We value your time, and you should too.  If you reach the 2 hour mark, save your progress and we can discuss what you were able to accomplish. 

The theory portions of this test are more open-ended.  It is up to you how much time you spend addressing these questions.  We recommend spending less than 1 hour.  


For the record, we are not testing to see how much free time you have, so there will be no extra credit for monumental time investments.  We are looking for concise, clear answers that demonstrate domain expertise.

# Project Overview
This project is a simple game database and consists of 2 components.  

The first component is a VueJS UI that communicates with an API and renders data in a simple browser-based UI.

The second component is an Express-based API server that queries and delivers data from an SQLite data source, using the Sequelize ORM.

This code is not necessarily representative of what you would find in a Voodoo production-ready codebase.  However, this type of stack is in regular use at Voodoo.

# Project Setup
You will need to have Node.js, NPM, and git installed locally.  You should not need anything else.

To get started, initialize a local git repo by going into the root of this project and running `git init`.  Then run `git add .` to add all of the relevant files.  Then `git commit` to complete the repo setup.  You will send us this repo as your final product.
  
Next, in a terminal, run `npm install` from the project root to initialize your dependencies.

Finally, to start the application, navigate to the project root in a terminal window and execute `npm start`

You should now be able to navigate to http://localhost:3000 and view the UI.

You should also be able to communicate with the API at http://localhost:3000/api/games

If you get an error like this when trying to build the project: `ERROR: Please install sqlite3 package manually` you should run `npm rebuild` from the project root.

# Practical Assignments
Pretend for a moment that you have been hired to work at Voodoo.  You have grabbed your first tickets to work on an internal game database application. 

#### FEATURE A: Add Search to Game Database
The main users of the Game Database have requested that we add a search feature that will allow them to search by name and/or by platform.  The front end team has already created UI for these features and all that remains is for the API to implement the expected interface.  The new UI can be seen at `/search.html`

The new UI sends 2 parameters via POST to a non-existent path on the API, `/api/games/search`

The parameters that are sent are `name` and `platform` and the expected behavior is to return results that match the platform and match or partially match the name string.  If no search has been specified, then the results should include everything (just like it does now).

Once the new API method is in place, we can move `search.html` to `index.html` and remove `search.html` from the repo.

#### FEATURE B: Populate your database with the top 100 apps
Add a populate button that calls a new route `/api/games/populate`. This route should populate your database with the top 100 games in the App Store and Google Play Store.
To do this, our data team have put in place 2 files at your disposal in an S3 bucket in JSON format:

- https://wizz-technical-test-dev.s3.eu-west-3.amazonaws.com/ios.top100.json
- https://wizz-technical-test-dev.s3.eu-west-3.amazonaws.com/android.top100.json

# Theory Assignments
You should complete these only after you have completed the practical assignments.

The business goal of the game database is to provide an internal service to get data for all apps from all app stores.  
Many other applications at Voodoo will use consume this API.

#### Question 1:
We are planning to put this project in production. According to you, what are the missing pieces to make this project production ready? 
Please elaborate an action plan.

##### Security

To deploy this project to production, the API must implement proper authentication. I recommend using JWTs. Role-based access control should also be implemented, so that some users have read-only permissions while others can perform read/write operations. The API should also be able to track who is accessing or modifying data.

When using JWTs, the API must handle user login and return two tokens to the client (whether via a UI or directly through the API): an access token and a refresh token. The access token should have a short lifespan, while the refresh token should last longer. This approach is secure but can be cumbersome in backend-to-backend scenarios. For such cases, many APIs provide fixed tokens with long or no expiration. Fixed tokens should only be used for fully backend integrations. Any frontend application must use the access/refresh token approach.

The application and API must be deployed on servers accessible exclusively via HTTPS. CORS should be configured to restrict access to trusted origins.

Brute-force protection may not be strictly necessary since this is an internal tool, but it could be beneficial if JWTs are used, particularly to protect the login route from repeated attacks. Rate limiting can be implemented using express-rate-limit.

We must not trust data coming from client requests. All input should be validated and sanitized, for example using libraries like Zod. Regarding SQL Injection, Sequelize provides protection through parameterized queries, but input validation remains important.

All dependencies should be kept up to date. Currently, there are 3 reported vulnerabilities (2 moderate, 1 critical). In particular, the current version of Sequelize contains critical vulnerabilities and should be updated promptly.


##### Database

Currently, the database is a simple local file. A production-grade application requires a proper database (e.g., PostgreSQL, MariaDB) secured on an appropriate server. The database must be regularly backed up to prevent data loss and ensure quick restoration in case of failure.

##### Monitoring

For a production application, we need to have monitoring and error detections. This can be handled with tools like Datadog.


#### Question 2:
Let's pretend our data team is now delivering new files every day into the S3 bucket, and our service needs to ingest those files
every day through the populate API. Could you describe a suitable solution to automate this? Feel free to propose architectural changes.

Currently, the "populate" button is for demonstration purposes only. In production, the database should be updated automatically and periodically. Recommended approaches include:

Scheduled updates: a nightly cron job to import new data.

Event-driven updates: leveraging S3 events, a Lambda function can automatically update the database whenever a new file is uploaded, minimizing delays.

Additionally, the system should implement duplicate checks. If the primary key is the application ID, any attempt to insert an existing application should be ignored.


