# atlas-heroku-deploy-steps
heroku / atlas mongoDb deploy instructions


You Do: Deploy Book-e JSON
Today, we will be deploying the 'solution' branch of our Book-e JSON excercise. You can clone it here.

Note:
When you clone a repo, it will clone down the master branch. Make sure you git checkout solution to switch to the solution branch once you've cloned!
Heroku
We'll be using a service called Heroku to deploy our apps, because it makes all the steps for deployment easy, simplifying and expediting the process. For example, Heroku automatically does the following...

Starts up a new server when we run heroku create, and installs all the necessary services
Adds a new remote to our Git repo, so we can just run git push heroku master to copy our code over
Detects our database
Detects the language our program is written in and chooses a buildpack
Automatically installs our app's dependencies, and starts our app
Easily change configuration information using heroku config
Heroku also has a FREE pricing tier!

Mongo Atlas
Mongo Atlas is a cloud-based database service that hosts a protected MongoDB instance that you can easily integrate with an application deployed on Heroku.

Today, you will need to set up an Atlas account and database to host our book-e database.

Deployment in 19 Easy Steps
Deployment is essentially an exercise in following directions. Follow the step-by-step instructions below to deploy your Book-e JSON App. Pay attention to the notes following each prompt! Fully read each prompt (including the notes) before executing each step.

Set up Heroku
Sign up for a free Heroku account.

Follow the instructions here to download the Heroku CLI.

NOTE: In your terminal, you will run the command brew tap heroku/brew && brew install heroku

From your project directory, create an app on Heroku

$ heroku create <your-app-name>
NOTE:
Make sure you are in your Book-e JSON App project directory before you run this command

NOTE: heroku create prepares Heroku to receive your code. Heroku will randomly create an app name for you if you don't specify one. You may need to login before you can create an app.

NOTE: If you choose to name your application, you will need to use a unique name (something someone else has not used before). If the name is taken, Heroku will prompt you to choose something new.

NOTE: Running into this error? ! Unable to connect to Heroku API, please check internet connectivity and try again. See how to debug: https://help.heroku.com/GOLWIGSP/why-can-t-i-connect-or-authenticate-with-the-heroku-command-line-cli

Set up MongoDB Atlas
Go to Mongo Atlas and sign up for an account, or sign in if you have one already.

In the sidebar on the left, open the dropdown menu title context and select New Project. Give it a name and click Create Project. You can leave the defaults in the "Create a Project" page and hit the "Create Project" button.

Create a project

Finish creating your new project and click the Build a Cluster button. Be sure to select the free option for building your cluster! Otherwise, leave all the default settings. Your cluster could take a few minutes to to finish building.

Build a cluster

When your cluster is finished, click the "Connect" button.

Add 0.0.0.0/0 for the whitelisted IP address, If you just add the current IP address you can click on "Network Access" in the sidebar, then the "Add IP Address" button in the top right, and finally "Allow Access from Anywhere" and click Confirm.

NOTE:
If you forget to whitelist the IP, go to "Network Access" under "Security" on the left sidebar. Next select "Add IP Address" in the top right corner, and there should be a "Allow Access from Anywhere" button (or you can enter 0.0.0.0/0 manually).
Also create a username. For the password, please use an auto-generated password. Remember the username and password you use for your database, you'll need them in a later step!

Go to "Database Access" under "Security" on the left sidebar. Then hit the "Add New User" button on the top right.

NOTE:
This is not the user with which you logged in to Atlas. "User" refers to an app that has access to your database, and not your Atlas account/username.

Create a Database username and Database password that you will remember, or write it down somewhere. You will need this information again later.

Do not use any special characters! Special characters can complicate the process when configuring your Atlas database with Heroku.

Do not check 'Make read-only'. Full CRUD functionality will not work with a read-only database. Making this user admin is perfect!

Heroku & Node Setup
Next we need to let our Node app know when to use Mongo Atlas as our database, and when to use our local DB.

A built-in environment variable, NODE_ENV available under process.env.NODE_ENV, is used to define the application environment. When a Node app is deployed to Heroku, Heroku automatically sets the NODE_ENV variable to 'production'.

The code below is stating that we should use the Mongo Atlas URI (in other words, the link that connects us to the Atlas database) when in production, and the local database at all other times.

In your Node app's db/connection.js file, we want to use the environment to determine the mongoURI for our application to connect to.
```

- const mongoURI = "mongodb://localhost/book-e"
+ let mongoURI = "";
```
Then, add
```
if (process.env.NODE_ENV === "production") {
  mongoURI = process.env.DB_URL;
} else {
  mongoURI = "mongodb://localhost/book-e";
}
The mongoose.connect method will stay the same.

```
NOTE:
In the example above, the link to the MongoDB includes the name of the database we are using, which in this case, is book-e. When using a different database for your own projects, make sure you include the name of the actual database you want to connect.
Next, you will need to make a minor change to index.js. When Heroku starts your app it will automatically assign a port to process.env.PORT (an environmental variable!) to be used in production. We can modify app.listen to accomodate Heroku's production port and our own local development port. Add the following to index.js...
```
app.set("port", process.env.PORT || 8080);

app.listen(app.get("port"), () => {
  console.log(`✅ PORT: ${app.get("port")} 🌟`);
});
```
Heroku looks for instruction when starting your app. In this case, that instruction is to run node index.js. In package.json under scripts, add the following...
```
"scripts": {
+    "start": "node index.js"
}
```
Note:
NOTE: Another way to do this is to define a Procfile in the root of your directory and include the line web: node index.js.
Add and commit all changes we've made to Book-e JSON. It is a good idea to push to GHE as well. IMPORTANT if you used the solution branch, make sure you're making the commit to the solution branch.

Heroku & Atlas Configuration
Go back to your Atlas database in your browser. Click on "Connect" and then the "Choose a Connection Method" button, then "Connect Your Application", then "Short SRV Connection String". Copy the connection string!



Note:
You must copy this from your own database to capture your unique database id numbers.

You will still need to manually substitute the USERNAME and PASSWORD with the one you created in the next step.

Set the URI you just copied as an environment variable called DB_URL using heroku config:set, filling in the <USERNAME> and <PASSWORD> you created on the "Users" page. Run the following in your terminal (in your project's folder)...
```
$ heroku config:set DB_URL="mongodb+srv://<USERNAME>:<PASSWORD>@cluster0-8mjuz.mongodb.net/test?retryWrites=true"

# CONFIRM
$ heroku config
```

Note:
Do not copy the above command. The database id numbers in the URI should match the URI that you copied from your own Atlas database. The sample command above includes id numbers of 012345 and 12345. Does your own Atlas URI match this? Probably not.

Your database name will be included in the URI you copied from your Atlas database. You will need to manually add the USERNAME and PASSWORD that you created in Step 10. DONT FORGET TO PUT QUOTES AROUND THE URL PART

Assigning environmental variables using heroku config:set is very similar to using export, the difference being accessibility. Variables assigned using the heroku command are only accessible from the production app deployed on heroku.

Deploying to Heroku
Push your code to Heroku remote. Because you are on the solution branch of the Book-e JSON App, you will need to run $ git push heroku solution:master in your terminal. This ensures that your most up-to-date code -- a.k.a. our solution branch is deployed.

Note:
If you are deploying to Heroku from the master branch, you can run the command 
```
$ git push heroku master.
```
Seed your Atlas database by running the command:
```
$ heroku run node db/seed.js
```

Note:
heroku run allows you to run js files on the heroku server. We can seed our database on heroku using the same seed file we used locally.
Open your application! Run the command 
```
$ heroku open 
```
in your terminal. This will launch your production app in a new browser tab.

You just successfully deployed your first app! You should be proud, so pat yourself on the back, give your neighbor a high five, call your parents, and share this milestone with someone you love!
