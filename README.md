# How to deploy a Strapi app on Railway

This article will showcase how you can deploy a Strapi app on Railway. It will look at deploying the Strapi app using the default SQLite database and then look at deploying using the Railway provided PostgreSQL database. It will also look at how you can make file storage persist by attaching a persistent disk to your Strapi app. It will also look at how you can save costs by enabling serverless mode in Railway's dashboard. The final part of the article will be a comparison of deploying your Strapi app on Railway versus deploying it on Strapi Cloud.

## Prerequisites

To follow along with this article you will need the following:
- Node.js installed on your machine (LTS versions 18, 20 or 22). [Download Node.js here](https://nodejs.org/download)
- Railway account. [Create a Railway Account here](https://railway.com)
- Some familiarity with Strapi. Check out the [Strapi Quickstart tutorial](https://docs.strapi.io/cms/quick-start) to get up to speed with Strapi

## Deploy to Railway using a SQLite database

### Scaffold a Strapi project locally

On your local machine, open your terminal in your work directory and create a Strapi project named `my-app` using the following command:
```shell
npx create-strapi-app@latest my-app
```

Answer the prompts as follows:
```
Ok to proceed? (y) y
? Please log in or sign up. Skip
? Do you want to use the default database (sqlite) ? No
? Choose your default database client sqlite
? Filename: .tmp/data.db
? Start with an example structure & data? No
? Start with Typescript? No
? Install dependencies with npm? Yes
? Initialize a git repository? No
```

After answering the prompts, wait for your Strapi app to be installed.

Navigate into your Strapi app folder, `my-app`:
```shell
cd my-app
```

### Create an Admin

Create an admin user for your Strapi app using the following command. The email for the admin is `chef@strapi.io` and the password for the admin is `Gourmet1234`.
```shell
npm run strapi admin:create-user -- --firstname=Kai --lastname=Doe --email=chef@strapi.io --password=Gourmet1234
```

Start your Strapi server using the following command:
```shell
npm run develop
```

This will start your Strapi server on port 1337. Visit `localhost:1337/admin` in your browser. Login using the credentials for the admin user you created earlier.

![Strapi Admin Login page](https://res.cloudinary.com/craigsims808/image/upload/v1752494792/strapi/stprwy/strapi-admin-login_lxxm9r.png)

### Create a "Post" collection type

Visit the [Content-Type-Builder page](http://localhost:1337/admin/plugins/content-type-builder/content-types/create-content-type).

![Content-Type Builder](https://res.cloudinary.com/craigsims808/image/upload/v1752495885/strapi/stprwy/content-type-builder_n9hket.png)

Click **+** next to **COLLECTION TYPES** to create a new collection type named `Post`.

Add the following fields to the `Post` collection:
- `title`: a Text field (short text)
- `content`: a Text field (long text)
- `image`: a Media field (single media)

![Post collection fields](https://res.cloudinary.com/craigsims808/image/upload/v1752504180/strapi/stprwy/post-collection_izql4q.png)

Click **Save** and wait for your server to restart.

### Install Railway CLI tool

After your `Post` collection has been created, you can move onto deploying your Strapi app to Railway.

In your terminal, stop your Strapi server by pressing the keyboard combo `ctrl` + `c`.

Raiway provides a CLI tool to help deploy your app. This is what you will use to deploy your Strapi app.

Install the Railway CLI by running the following command:
```shell
npm i -g @railway/cli
```

### Authenticate your Railway account

Login to your Railway account with this command:
```shell
railway login
```
This command opens a new tab in your default browser to the https://railway.com authentication page. Follow the instructions to complete the authentication process.

### Create and Link Railway project

Create a new Railway project and give it a name. In this case, I named mine `my-app`. Run this in your terminal:
```shell
railway init --name my-app
```

Make sure you are in your Strapi project folder, `my-app` and run the following command to link your Strapi project folder to your Railway project named `my-app`:
```shell
railway link --project my-app
```

### Create and Link Railway service

Create a new Railway service in your Railway project and give it a name. I named mine `strapi`. Run this command:
```shell
railway add --service strapi
```

Link the newly created Railway service to your Railway project you created previously:
```shell
railway link --service strapi
```

### Add environment variables to Railway service

Copy the contents of your `.env` file in your local Strapi project folder. You will add them to your Railway service as environment variables.

Add environment variables to your Railway service using the following command:
```shell
railway variables --set "HOST=0.0.0.0" --set "PORT=1337" --set "APP_KEYS=N8gCsMYVvZpLxDgiKc3MuA==,HDfOEXIbz3HR/XOpbYUh/g==,YtS15s72lk0NtVFopNwpIg==,OpcmnTK5yzRU0/j/SR/tUA==" --set "API_TOKEN_SALT=u7an+XYMBH+axaGfKbIBmw==" --set "ADMIN_JWT_SECRET=UE0aC/TV8uC1SloxnP+cbw==" --set "TRANSFER_TOKEN_SALT=U3K+3NSe/brqtPvHDeElbw==" --set "ENCRYPTION_KEY=8QOTqBTVpk7a1ycb4DAUDg==" --set "DATABASE_CLIENT=sqlite" --set "DATABASE_SSL=false" --set "DATABASE_FILENAME=.tmp/data.db" --set "JWT_SECRET=0YEEd8vLFu/RvoQcuAbEHA=="
```

<!--
Alternatively, use this command:
```shell
railway add --service --variables "HOST=0.0.0.0" --variables "PORT=1337" --variables "APP_KEYS=N8gCsMYVvZpLxDgiKc3MuA==,HDfOEXIbz3HR/XOpbYUh/g==,YtS15s72lk0NtVFopNwpIg==,OpcmnTK5yzRU0/j/SR/tUA==" --variables "API_TOKEN_SALT=u7an+XYMBH+axaGfKbIBmw==" --variables "ADMIN_JWT_SECRET=UE0aC/TV8uC1SloxnP+cbw==" --variables "TRANSFER_TOKEN_SALT=U3K+3NSe/brqtPvHDeElbw==" --variables "ENCRYPTION_KEY=8QOTqBTVpk7a1ycb4DAUDg==" --variables "DATABASE_CLIENT=sqlite" --variables "DATABASE_SSL=false" --variables "DATABASE_FILENAME=.tmp/data.db" --variables "JWT_SECRET=0YEEd8vLFu/RvoQcuAbEHA=="
```
-->

### Add Config-as-code using `railway.toml`

Create a `railway.toml` file in the root of your Strapi project folder:
```toml
[build]
builder = "NIXPACKS"

[build.nixpacksPlan.phases.setup]
nixPkgs = ["nodejs_22"]

[build.nixpacksPlan.phases.install]
cmds = ["npm install"]

[build.nixpacksPlan.phases.build]
cmds = ["npm run build"]

[deploy]
startCommand = "npm run start"
```

### Deploy app and generate domain

Deploy your Strapi app to Railway using the following command:
```shell
railway up
```

This will upload your Strapi project folder to Railway's servers. Wait for your project to be built and then deployed.

Once the deployment is ready, generate a domain for your Strapi app using the following command:
```shell
railway domain
```

Visit the domain in your browser and register an admin user for your Strapi production app.
![Register admin in Strapi hosted on Railway](https://res.cloudinary.com/craigsims808/image/upload/v1752528391/strapi/stprwy/strapi-register-admin-railway_orgjvh.png)

### Add an entry to your "Post" collection

Once logged in to your Strapi Admin dashboard, create an entry for your "Post" collection. Use this sample text to create an entry:

In the *title* field, add `Strapi 5 Rocks`. 

In the *content* field, add the following text:
```
Strapi 5 truly rocks. It's a massive leap forward packed with powerful new features like reworked Draft & Publish and Content History, making CMS workflows feel smooth and intuitive. Under the hood, a full TypeScript rewrite, Vite-powered builds, and a refined API (both REST & GraphQL) keep things blazing fast and developer-friendly. Plus, the introduction of a sleek Plugin SDK, powerful Document Service API, and automated upgrade CLI make extending and evolving Strapi projects a breeze 
```

In the *Image* field download the image from the following [link](https://res.cloudinary.com/craigsims808/image/upload/v1752529602/strapi/stprwy/Strapi.vertical.logo.dark_lplqna.png) and upload it.

Click **Save** to save your entry.

### Set Roles and Permissions for "Post" collection

After adding an entry to your "Post" collection, next set roles and permissions to make the content accessible.

Visit the **Settings** page then select **Roles** under **Users & Permissions Plugin**.

![Users and Permissions Plugin Roles Menu](https://res.cloudinary.com/craigsims808/image/upload/v1752530329/strapi/stprwy/roles-under-user-permissions_zfzafx.png)

Select **Public** role. Click **Permissions** tab and select "Post".

Enable the `find` and `findOne` permission for your "Post" collection.

![Post Collection Permissions](https://res.cloudinary.com/craigsims808/image/upload/v1752530621/strapi/stprwy/post-permissions_aorw79.png)

Click **Save** to save your API permissions and make it publicly accessible.

### Use the API

Visit `<your-railway-project-url>/api/posts` in your browser to access the list of posts from Strapi API backend. (Where `<your-railway-project-url>` is the URL of your Railway hosted Strapi project.)

You should get an API response similar to the screenshot below:
![API response](https://res.cloudinary.com/craigsims808/image/upload/v1752531130/strapi/stprwy/api-response_o0tshr.png)

The databases for your Railway hosted project and your local project are different. This means that data is not automatically synchronized between your Railway project and local projects. You can use the [data management system](https://docs.strapi.io/cms/features/data-management) to transfer data between projects.

## Deploy to Railway using a PostgreSQL database

- Install Railway CLI
- Login Railway
- Create Railway project
- Create Railway Postgres service in newly created Railway project
- Retrieve data (`DATABASE_CLIENT`,`DATABASE_HOST`,`DATABASE_PORT`,`DATABASE_NAME`,`DATABASE_PASSWORD`,`DATABASE_SSL`,`DATABASE_FILENAME`,`DATABASE_USERNAME`)
- Create Strapi project (configured for Postgres)
- Create admin
- Login
- Create "Post" collection
- Stop Strapi server
- Link Railway project to Strapi folder
- Create and Link Railway service
- Add env var
- Add config-as-code
- Deploy and generate domain
- Add an entry to "Post collection"
- Set Roles and Permissions
- Use the API

## How to make file storage persist for your Strapi app

- Assuming project is already deployed

## Cost saving measures

- Serverless mode
- Use file storage provider like Cloudinary instead of attaching a persistent disk
- Split hosting Strapi. Backend on Railway. Frontend on Vercel or some other static host.

## Strapi Cloud vs Railway

- Strapi Cloud has free tier (no credit card required). Railway has free trial which expires.
- Strapi Cloud has built in email. Railway you have to use third party services like Sendgrid.
- Strapi Cloud has built in persistent storage for your projects. Railway you have to attach a persistent disk at an extra cost.
- Beginner friendly
- Knowledgeable support team

- Railway has got flexibility, you can host in serverless mode to save costs or split deploy Strapi to save costs as well.
- Case to use Railway is when you have already paid for Railway and want to benefit from the ecosystem
- Most cases use Strapi Cloud. 