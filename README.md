# How to deploy a Strapi app on Railway

This article will showcase how you can deploy a Strapi app on Railway. It will look at deploying the Strapi app using the default SQLite database and then look at deploying using the Railway provided PostgreSQL database. It will also look at how you can make file storage persist by attaching a persistent disk to your Strapi app. It will also look at how you can save costs by enabling serverless mode in Railway's dashboard. The final part of the article will be a comparison of deploying your Strapi app on Railway versus deploying it on Strapi Cloud.

## Prerequisites

To follow along with this article you will need the following:
- Node.js installed on your machine (LTS versions 18, 20 or 22). [Download Node.js here](https://nodejs.org/download)
- Railway account. [Create a Railway Account here](https://railway.com)
- Some familiarity with Strapi. Check out the [Strapi Quickstart tutorial](https://docs.strapi.io/cms/quick-start) to get up to speed with Strapi

## Deploy to Railway using a SQLite database

This part looks at how you can deploy Strapi to Railway using a SQLite database. If you prefer a deployment done using a PostgreSQL database, please skip to the part titled: "Deploy to Railway using a PostgreSQL database".

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

### Create an Admin and Login

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

Railway provides a CLI tool to help deploy your app. This is what you will use to deploy your Strapi app.

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

After running the command, a link to your project is displayed in the terminal.

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
railway link --service strapi --project my-app
```

### Add environment variables to Railway service

Copy the contents of your `.env` file in your local Strapi project folder. You will add them to your Railway service as environment variables.

Add environment variables to your Railway service using the following command:
```shell
railway variables --set "HOST=0.0.0.0" --set "PORT=1337" --set "APP_KEYS=N8gCsMYVvZpLxDgiKc3MuA==,HDfOEXIbz3HR/XOpbYUh/g==,YtS15s72lk0NtVFopNwpIg==,OpcmnTK5yzRU0/j/SR/tUA==" --set "API_TOKEN_SALT=u7an+XYMBH+axaGfKbIBmw==" --set "ADMIN_JWT_SECRET=UE0aC/TV8uC1SloxnP+cbw==" --set "TRANSFER_TOKEN_SALT=U3K+3NSe/brqtPvHDeElbw==" --set "ENCRYPTION_KEY=8QOTqBTVpk7a1ycb4DAUDg==" --set "DATABASE_CLIENT=sqlite" --set "DATABASE_SSL=false" --set "DATABASE_FILENAME=.tmp/data.db" --set "JWT_SECRET=0YEEd8vLFu/RvoQcuAbEHA=="
```

> **NOTE:**
>
> *Replace the variables with your own Strapi project variables from your .env file*

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

Once the build is complete. Return to your terminal using the `ctrl` + `c` keyboard combo.

Generate a domain for your Strapi app using the following command:
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

This part looks at how you can deploy Strapi to Railway using a PostgreSQL database. If you have already followed along the previous part "Deploy to Railway using a SQLite database", you can skip to the next part "How to make file storage persist for your Strapi app"

### Install Railway CLI tool

Railway provides a CLI tool to help deploy your app. Install the Railway CLI tool by running the following command in your terminal:
```shell
npm i -g @railway/cli
```

### Authenticate your Railway account

Login to your Railway account with this command:
```shell
railway login
```

This command opens a new tab in your default browser to the https://railway.com authentication page. Follow the instructions to complete the authentication process.

### Create Railway project

Create a new Railway project and give it a name. In this case, I named mine `my-app-pg`. Run this in your terminal:
```shell
railway init --name my-app-pg
```

After running the command, a link to your project is displayed in the terminal.

### Create Railway Postgres service

Create a new Railway service to host the Potgres database in your Railway project. Run the following command:
```shell
railway add --database postgres
```

### Retrieve Database Details

The next step requires retrieving the details of your newly created postgres database service.

Run the following command in your terminal:
```shell
railway variables --service Postgres
```

The output of this command, should be similar to this:
```
variables --service Postgres
╔═══════════════════════════ Variables for Postgres ═══════════════════════════╗
║ DATABASE_PUBLIC_URL                 │ postgresql://                          ║
║                                     │ postgres:FterodplxexPuubzTwxCFKktFCdFJ ║
║                                     │ d0t@caboose.proxy.rlwy.net:26362/      ║
║                                     │ railway                                ║
║──────────────────────────────────────────────────────────────────────────────║
║ DATABASE_URL                        │ postgresql://                          ║
║                                     │ postgres:FterodplxexPuubzTwxCFKktFCdFJ ║
║                                     │ d0t@postgres.railway.internal:5432/    ║
║                                     │ railway                                ║
║──────────────────────────────────────────────────────────────────────────────║
║ PGDATA                              │ /var/lib/postgresql/data/pgdata        ║
║──────────────────────────────────────────────────────────────────────────────║
║ PGDATABASE                          │ railway                                ║
║──────────────────────────────────────────────────────────────────────────────║
║ PGHOST                              │ postgres.railway.internal              ║
║──────────────────────────────────────────────────────────────────────────────║
║ PGPASSWORD                          │ FterodplxexPuubzTwxCFKktFCdFJd0t       ║
║──────────────────────────────────────────────────────────────────────────────║
║ PGPORT                              │ 5432                                   ║
║──────────────────────────────────────────────────────────────────────────────║
║ PGUSER                              │ postgres                               ║
║──────────────────────────────────────────────────────────────────────────────║
║ POSTGRES_DB                         │ railway                                ║
║──────────────────────────────────────────────────────────────────────────────║
║ POSTGRES_PASSWORD                   │ FterodplxexPuubzTwxCFKktFCdFJd0t       ║
║──────────────────────────────────────────────────────────────────────────────║
║ POSTGRES_USER                       │ postgres                               ║
║──────────────────────────────────────────────────────────────────────────────║
║ RAILWAY_DEPLOYMENT_DRAINING_SECONDS │ 60                                     ║
║──────────────────────────────────────────────────────────────────────────────║
║ RAILWAY_ENVIRONMENT                 │ production                             ║
║──────────────────────────────────────────────────────────────────────────────║
║ RAILWAY_ENVIRONMENT_ID              │ 641505af-03c9-447c-919e-21bb82b4c2fa   ║
║──────────────────────────────────────────────────────────────────────────────║
║ RAILWAY_ENVIRONMENT_NAME            │ production                             ║
║──────────────────────────────────────────────────────────────────────────────║
║ RAILWAY_PRIVATE_DOMAIN              │ postgres.railway.internal              ║
║──────────────────────────────────────────────────────────────────────────────║
║ RAILWAY_PROJECT_ID                  │ 3f96574a-9920-43fb-b09a-5b86c63a486f   ║
║──────────────────────────────────────────────────────────────────────────────║
║ RAILWAY_PROJECT_NAME                │ my-app-pg                              ║
║──────────────────────────────────────────────────────────────────────────────║
║ RAILWAY_SERVICE_ID                  │ b1cd1deb-5fe5-4c33-b588-6244abd991bd   ║
║──────────────────────────────────────────────────────────────────────────────║
║ RAILWAY_SERVICE_NAME                │ Postgres                               ║
║──────────────────────────────────────────────────────────────────────────────║
║ RAILWAY_TCP_APPLICATION_PORT        │ 5432                                   ║
║──────────────────────────────────────────────────────────────────────────────║
║ RAILWAY_TCP_PROXY_DOMAIN            │ caboose.proxy.rlwy.net                 ║
║──────────────────────────────────────────────────────────────────────────────║
║ RAILWAY_TCP_PROXY_PORT              │ 26362                                  ║
║──────────────────────────────────────────────────────────────────────────────║
║ RAILWAY_VOLUME_ID                   │ 4e07339d-6aaf-4fdb-8a45-3741a10ec50c   ║
║──────────────────────────────────────────────────────────────────────────────║
║ RAILWAY_VOLUME_MOUNT_PATH           │ /var/lib/postgresql/data               ║
║──────────────────────────────────────────────────────────────────────────────║
║ RAILWAY_VOLUME_NAME                 │ spirited-volume                        ║
║──────────────────────────────────────────────────────────────────────────────║
║ SSL_CERT_DAYS                       │ 820                                    ║
╚══════════════════════════════════════════════════════════════════════════════╝
```

<!--
The output of this command, should be similar to this:
```
variables --service Postgres
╔═══════════════════════════ Variables for Postgres ═══════════════════════════╗
║ DATABASE_PUBLIC_URL                 │ postgresql://                          ║
║                                     │ postgres:YixGoYplxexPuubzTwxCFKktFCdFJ ║
║                                     │ xIa@caboose.proxy.rlwy.net:26362/      ║
║                                     │ railway                                ║
║──────────────────────────────────────────────────────────────────────────────║
║ DATABASE_URL                        │ postgresql://                          ║
║                                     │ postgres:YixGoYplxexPuubzTwxCFKktFCdFJ ║
║                                     │ xIa@postgres.railway.internal:5432/    ║
║                                     │ railway                                ║
║──────────────────────────────────────────────────────────────────────────────║
║ PGDATA                              │ /var/lib/postgresql/data/pgdata        ║
║──────────────────────────────────────────────────────────────────────────────║
║ PGDATABASE                          │ railway                                ║
║──────────────────────────────────────────────────────────────────────────────║
║ PGHOST                              │ postgres.railway.internal              ║
║──────────────────────────────────────────────────────────────────────────────║
║ PGPASSWORD                          │ YixGoYplxexPuubzTwxCFKktFCdFJxIa       ║
║──────────────────────────────────────────────────────────────────────────────║
║ PGPORT                              │ 5432                                   ║
║──────────────────────────────────────────────────────────────────────────────║
║ PGUSER                              │ postgres                               ║
║──────────────────────────────────────────────────────────────────────────────║
║ POSTGRES_DB                         │ railway                                ║
║──────────────────────────────────────────────────────────────────────────────║
║ POSTGRES_PASSWORD                   │ YixGoYplxexPuubzTwxCFKktFCdFJxIa       ║
║──────────────────────────────────────────────────────────────────────────────║
║ POSTGRES_USER                       │ postgres                               ║
║──────────────────────────────────────────────────────────────────────────────║
║ RAILWAY_DEPLOYMENT_DRAINING_SECONDS │ 60                                     ║
║──────────────────────────────────────────────────────────────────────────────║
║ RAILWAY_ENVIRONMENT                 │ production                             ║
║──────────────────────────────────────────────────────────────────────────────║
║ RAILWAY_ENVIRONMENT_ID              │ 641505af-03c9-447c-919e-21bb82b4c2fa   ║
║──────────────────────────────────────────────────────────────────────────────║
║ RAILWAY_ENVIRONMENT_NAME            │ production                             ║
║──────────────────────────────────────────────────────────────────────────────║
║ RAILWAY_PRIVATE_DOMAIN              │ postgres.railway.internal              ║
║──────────────────────────────────────────────────────────────────────────────║
║ RAILWAY_PROJECT_ID                  │ 3f96574a-9920-43fb-b09a-5b86c63a486f   ║
║──────────────────────────────────────────────────────────────────────────────║
║ RAILWAY_PROJECT_NAME                │ my-app-pg                              ║
║──────────────────────────────────────────────────────────────────────────────║
║ RAILWAY_SERVICE_ID                  │ b1cd1deb-5fe5-4c33-b588-6244abd991bd   ║
║──────────────────────────────────────────────────────────────────────────────║
║ RAILWAY_SERVICE_NAME                │ Postgres                               ║
║──────────────────────────────────────────────────────────────────────────────║
║ RAILWAY_TCP_APPLICATION_PORT        │ 5432                                   ║
║──────────────────────────────────────────────────────────────────────────────║
║ RAILWAY_TCP_PROXY_DOMAIN            │ caboose.proxy.rlwy.net                 ║
║──────────────────────────────────────────────────────────────────────────────║
║ RAILWAY_TCP_PROXY_PORT              │ 26362                                  ║
║──────────────────────────────────────────────────────────────────────────────║
║ RAILWAY_VOLUME_ID                   │ 4e07339d-6aaf-4fdb-8a45-3741a10ec50c   ║
║──────────────────────────────────────────────────────────────────────────────║
║ RAILWAY_VOLUME_MOUNT_PATH           │ /var/lib/postgresql/data               ║
║──────────────────────────────────────────────────────────────────────────────║
║ RAILWAY_VOLUME_NAME                 │ spirited-volume                        ║
║──────────────────────────────────────────────────────────────────────────────║
║ SSL_CERT_DAYS                       │ 820                                    ║
╚══════════════════════════════════════════════════════════════════════════════╝
```
-->

Copy the variables as you will use them later when creating a new Strapi project. The variables you should take note of are the following are `DATABASE_PUBLIC_URL` and `DATABASE_URL`.

The `DATABASE_PUBLIC_URL` will be used for the local Strapi installation. In my case it is `postgresql://postgres:FterodplxexPuubzTwxCFKktFCdFjd0t@caboose.proxy.rlwy.net:26362/railway`. It is in the form `postgresql://<Username>:<Password>@<Host>:<Port>/<Database name>`. Where:

- Username - `postgres`
- Password - `FterodplxexPuubzTwxCFKktFCdFjd0t`
- Host - `caboose.proxy.rlwy.net`
- Port - `26362`
- Database name - `railway`

<!--
postgresql://postgres:YixGoYplxexPuubzTwxCFKktFCdFJxIa@caboose.proxy.rlwy.net:26362/railway
-->

<!-- Retrieve data (`DATABASE_CLIENT`,`DATABASE_HOST`,`DATABASE_PORT`,`DATABASE_NAME`,`DATABASE_PASSWORD`,`DATABASE_SSL`,`DATABASE_FILENAME`,`DATABASE_USERNAME`) -->

### Scaffold a Strapi project locally

In your terminal, create a Strapi project named `my-app-pg` using the following command:
```shell
npx create-strapi-app@latest my-app-pg
```

Answer the prompts as follows:
```
Ok to proceed? (y) y
? Please log in or sign up. Skip
? Do you want to use the default database (sqlite) ? No
? Choose your default database client postgres
? Database name: railway
? Host: caboose.proxy.rlwy.net
? Port: 26362
? Username: postgres
? Password: FterodplxexPuubzTwxCFKktFCdFjd0t
? Enable SSL connection: No
? Start with an example structure & data? No
? Start with Typescript? No
? Install dependencies with npm? Yes
? Initialize a git repository? No
```

> **NOTE:**
>
> *Take note that the answers will depend on the `DATABASE_PUBLIC_URL` variable of your Railway hosted Postgres service*

After answering the prompts, wait for your Strapi app to be installed.

Navigate into your Strapi app folder, `my-app-pg`
```shell
cd my-app-pg
```

### Create admin user and login

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

### Link Railway project to local Strapi project

Return to your terminal and stop your local running Strapi server by entering the keyboard combo `ctrl` + `C`.

<!--
Install Railway CLI

```shell
npm i -g @railway/cli
```

Authenticate Railway
```shell
railway login --browserless
```
-->

Run the following command to link your Strapi project folder to your Railway project named `my-app-pg`:
```shell
railway link --project my-app-pg
```

### Create and Link Railway service

Create a new Railway service in your Railway project and give it a name. I named mine `strapi-pg`. Run this command:
```shell
railway add --service strapi-pg
```

Link the newly created Railway service to your Railway project, `my-app-pg` you created previously:
```shell
railway link --service strapi-pg --project my-app-pg
```

### Add environment variables to Railway service

Copy the contents of your `.env` file in your local Strapi project folder. Paste them to a temporary file.

```

```

<!-- Add link to verify claim -->
[Railway supports private networking](). This can save on egress fees when the Strapi service makes calls to the Postgres service. To take advantage of this feature we will use the `DATABASE_URL` variable instead of the `DATABASE_PUBLIC_URL` variable for configuring variables for the production version of our Strapi app.

Retrieve the `DATABASE_URL` variable from your Railway hosted Postgres service variables. In my case the `DATABASE_URL` is `postgresql://postgres:FterodplxexPuubzTwxCFKktFCdFJd0t@postgres.railway.internal:5432/railway`. 

<!--
postgresql://postgres:YixGoYplxexPuubzTwxCFKktFCdFJxIa@postgres.railway.internal:5432/railway 
-->

It is in the form `postgresql://<Username>:<Password>@<Host>:<Port>/<Database name>`. Where:
<!--Replace accordingly-->
- Username - `postgres`
- Password - `FterodplxexPuubzTwxCFKktFCdFjd0t`
- Host - `postgres.railway.internal`
- Port - `5432`
- Database name - `railway`

In the temporary file you created earlier, replace the values for the `DATABASE_HOST`, `DATABASE_NAME`, `DATABASE_USERNAME`, `DATABASE_PASSWORD` and `DATABASE_PORT` variables with the values from the `DATABASE_URL`. Therefore the changes become:
```
DATABASE_HOST=postgres.railway.internal
DATABASE_PORT=5432
DATABASE_NAME=railway
DATABASE_USERNAME=postgres
DATABASE_PASSWORD=FterodplxexPuubzTwxCFKktFCdFjd0t
```
<!--
DATABASE_HOST=postgres.railway.internal
DATABASE_PORT=5432
DATABASE_NAME=railway
DATABASE_USERNAME=postgres
DATABASE_PASSWORD=YixGoYplxexPuubzTwxCFKktFCdFJxIa
-->

Leave the other variables as is.

Add environment variables to your Railway service using the following command:
```shell
railway variables --set "HOST=0.0.0.0" --set "PORT=1337" --set "APP_KEYS=N8gCsMYVvZpLxDgiKc3MuA==,HDfOEXIbz3HR/XOpbYUh/g==,YtS15s72lk0NtVFopNwpIg==,OpcmnTK5yzRU0/j/SR/tUA==" --set "API_TOKEN_SALT=u7an+XYMBH+axaGfKbIBmw==" --set "ADMIN_JWT_SECRET=UE0aC/TV8uC1SloxnP+cbw==" --set "TRANSFER_TOKEN_SALT=U3K+3NSe/brqtPvHDeElbw==" --set "ENCRYPTION_KEY=8QOTqBTVpk7a1ycb4DAUDg==" --set "DATABASE_CLIENT=postgres" --set "DATABASE_HOST=postgres.railway.internal" --set "DATABASE_PORT=5432" --set "DATABASE_NAME=railway" --set "DATABASE_USERNAME=postgres" --set "DATABASE_PASSWORD=FterodplxexPuubzTwxCFKktFCdFjd0t" --set "DATABASE_SSL=false" --set "JWT_SECRET=0YEEd8vLFu/RvoQcuAbEHA=="
```

<!--
```shell
railway variables --set "HOST=0.0.0.0" --set "PORT=1337" --set "APP_KEYS=OBzFJ2b6M5b1Eq2Vn7002A==,y2ShiswEh4xsQSNcUF1xlg==,YtWcfA+tANayRqMoAFkeng==,xOM41STNM12kz3pynJLGJw==" --set "API_TOKEN_SALT=RoJ4+aZwwsDDvQnPvc7EfQ==" --set "ADMIN_JWT_SECRET=3lLwo386tqAlOUboNTxN/g==" --set "TRANSFER_TOKEN_SALT=vawH0OqvmjZ3//mG2v2t1Q==" --set "ENCRYPTION_KEY=WCQr9umjZgFQzQO9lz1XmQ==" --set "DATABASE_CLIENT=postgres" --set "DATABASE_HOST=postgres.railway.internal" --set "DATABASE_PORT=5432" --set "DATABASE_NAME=railway" --set "DATABASE_USERNAME=postgres" --set "DATABASE_PASSWORD=YixGoYplxexPuubzTwxCFKktFCdFJxIa" --set "DATABASE_SSL=false" --set "JWT_SECRET=VSEmI+jO6WX27xS9yO7Pww=="
```
-->

> **NOTE:**
>
> *Replace the variables with your own Strapi project variables from your .env file*


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

Once the build is complete. Return to your terminal using the `ctrl` + `c` keyboard combo.

Generate a domain for your Strapi app using the following command:
```shell
railway domain
```

Visit the domain in your browser and login using the admin user you created on your local Strapi installation:
![Login Strapi hosted on Railway](https://res.cloudinary.com/craigsims808/image/upload/v1752574622/strapi/stprwy/strapi-login-railway-pg_g8mqrs.png)

### Add an entry to your "Post" collection

Once logged in to your Strapi Admin dashboard, create an entry for your "Post" collection. Use this sample text to create an entry:

In the *title* field, add `Wow, Strapi Conf 2025`. 

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