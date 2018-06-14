# Installation

## HiveApi Application Installation

HiveApi can be installed automatically with Composer (recommended way) or manually (via Git or via direct download):

### 1) Download

In the following, both methods are described in short:

#### 1.A) Automatically via Composer

1) Clone the repo, install dependencies and setup the project:

Option 1: Latest [stable release](https://github.com/hiveapi/framework/releases/latest) release:

```shell
composer create-project hiveapi/framework my-awesome-api
```

Option 2: Target a [specific version](https://github.com/hiveapi/framework/releases):

```shell
composer create-project hiveapi/framework my-awesome-api ~major.minor
```

Option 3: Ongoing [development](https://github.com/hiveapi/framework/commits/master) branch `dev-master` (unstable):

> Heads up!
> 
> This may provide (unstable) features from the upcoming releases. You may need to keep syncing your project with 
the upstream `master` branch and run `composer update` in order to apply changes!* 

```shell
composer create-project hiveapi/framework my-awesome-api dev-master
```

2) Edit your `.env` variables to match with your environment (set database credentials, app url, ...).

3) Continue from [2) Database Setup](#Setup-Database) below.

#### 1.B) Manually

You can download the code directly from the repository as `.zip` file or clone the repository using `git` (recommended 
approach):

1) Clone the repository using `git`:

 ```shell
git clone https://github.com/hiveapi/framework.git
 ```

2) Install all dependency packages (including containers dependencies):

```shell
composer install
```

3) Create a new `.env` file by copying the provided `.env.example` file.

```shell
cp .env.example .env
```

> Heads up!
>
> heck all variables and edit accordingly!

4) Generate a random `APP_KEY`

```shell
php artisan key:generate
```

5) Delete the existing `.git` folder from the root directory and initialize your own one with `git init`.

### 2) Database Setup

1) Migrate the provided database by runing the migration artisan command:

```shell
php artisan migrate
```

2) Seed the database with the artisan command:

```shell
php artisan hive:seed:deploy
```

3) (optional) By default `HiveApi` seeds a "Super User", given the default `admin` role (the role has no permissions set 
to it). 

To give the `admin` role, access to all the seeded permissions in the system, run the following command at any time.
 
```shell
php artisan hive:permissions:toRole admin
```

If you are using Laradock, you need to run those commands from the `workspace` container, you can enter that container 
by running `docker-compose exec workspace bash` from the Laradock folder.

### 3) OAuth 2.0 Setup 

1) Create encryption keys to generate secure access tokens and create "personal access" and "password grant" clients, 
which will be used to generate access tokens for your users or applications:

```shell
php artisan passport:install
```

### 4) Documentation Setup

If you are planning to use ApiDoc JS then proceed with this setup, else skip this and use whatever you prefer:

1) Install [ApiDocJs](http://apidocjs.com/) using NPM or your favorite dependencies manager:

Install it Globally with `-g` or locally in the project without `-g` 

```shell
npm install apidoc -g
```

or install it by just running `npm install` on the root of the project, after checking the `package.json` file on the 
root. 

2) run `php artisan hive:docs`

Behind the scene `hive:docs` executes a command like this
```
apidoc -c app/Containers/Documentation/ApiDocJs/public -f public.php -i app -o public/api/documentation
```

See the [API Docs Generator](./../features/api-docs-generator) page for more details.

### 5) Tests Setup

1) Open `.env.testing` and set up the environment variables correctly.
 
2) Open the `/tests/_data/presets/*` files and adapt the `urls` accordingly to fit your domains.
 
3) Run the tests

```shell
vendor/bin/codecept run
```

<a name="Development-Environment"></a>
## B) Development Environment Setup

You can run `HiveApi` on your favorite environment. Below you see how you can run it on top of 
[Vagrant](https://www.vagrantup.com/) (using [Laravel Homestead](https://laravel.com/docs/master/homestead)) or 
[Docker](https://www.docker.com/) (using [Laradock](https://github.com/Laradock/laradock)). 

We will see how to use both tools and you can pick one, or you can use other options like 
[Larvel Valet](https://laravel.com/docs/valet), [Laragon](https://laragon.org/) or even run it directly on your machine.

> Heads up!
> 
> The ICANN has now officially approved `.dev` as a generic top level domain (gTLD). Therefore, it is **not** recommended 
> to use `.dev` domains any more in your local development setup! The docs here has been changed to use `.develop` 
> instead of `.dev`, however, you may change to `.localhost`, `.test`, or whatever suits your needs.

<a name="Dev-Env-Opt-A"></a>
### B.1) Using Docker (with Laradock)

`Laradock` is a Docker PHP development environment. It facilitate running PHP Apps on Docker.

1) Install [Laradock](https://github.com/LaraDock/laradock#installation).

2) Navigate into the `laradock` directory:

```shell
cd laradock
```
This directory contains a `docker-compose.yml` file. (From the `Laradock` project).

2.1) If you haven't done so, rename `env-example` to `.env`.

```shell
cp env-example .env
```

3) Run the Docker containers:

```shell
docker-compose up -d nginx mysql redis beanstalkd
```

4) Make sure you are setting the `Docker IP` as `Host` for the `DB` and `Redis`  in your `.env` file.

5) Add the domain to the `hosts` file:

5.1) Open the hosts file on your local machine `/etc/hosts`.

*We'll be using `hive.local` as local domain (you can change it if you want).*

5.2) Map the domain and its subdomains to 127.0.0.1:

```text
127.0.0.1  hive.local
127.0.0.1  api.hive.local
127.0.0.1  admin.hive.local
```

If you are using NGINX or Apache, make sure the **server_name** (in case of NGINX) or **ServerName** (in case of Apache) 
in your the server config file, is set to the following `hive.local api.hive.local admin.hive.local`. Also don't 
forget to set your **root** or **DocumentRoot** to the public directory inside hive (i.e., `hive/public`).

<a name="Dev-Env-Opt-B"></a>
### B.2) Using Vagrant (with Laravel Homestead)

1) Configure Homestead:

1.1) Open the Homestead config file:

```shell
homestead edit
```

1.2) Map the `api.hive.local` domain to the project public directory - Example:

```text
sites:
	- map: api.hive.local
  	  to: /{full-project-path}/hive/public
```

1.3) You can also map other domains like `hive.local` and `admin.hive.local` to other web apps:

```text
	- map: hive.local
  	  to: /{full-project-path}/clients/web/user
	- map: admin.hive.local
  	  to: /{full-project-path}/clients/web/admin
```

Note: in the example above the `/{full-project-path}/clients/web/xxx` are separate apps, who live in their own 
repositories and in different folder than the `HiveApi`. If your admins, users or other type of applications are 
within `HiveApi`, then you must point them all to the `HiveApi` project folder `/{full-project-path}/hive/public`. 
So in that case you would have something like this:
 
```text
    - map: api.hive.local
      to: /{full-project-path}/hive/public
    - map: hive.local
      to: /{full-project-path}/hive/public
    - map: admin.hive.local
      to: /{full-project-path}/hive/public
```

2) Add the domain to the hosts file:

2.1) Open the hosts file on your local machine `/etc/hosts`.

*We'll be using `hive.local` as local domain (you can change it if you want).*

2.2) Map the domain and its subdomains to the Vagrant IP Address:

```text
192.168.10.10   hive.local
192.168.10.10   api.hive.local
192.168.10.10   admin.hive.local
```

If you are using NGINX or Apache, make sure the **server_name** (in case of NGINX) or **ServerName** (in case of Apache) 
in your the server config file, is set to the following `hive.local api.hive.local admin.hive.local`. Also don't 
forget to set your **root** or **DocumentRoot** to the public directory inside hive (i.e., `hive/public`).

2.3) Run the Virtual Machine:

```shell
homestead up --provision
```

*If you see `No input file specified` on the sub-domains!  
try running this command `homestead halt && homestead up --provision`.*

<a name="Dev-Env-Opt-C"></a>
### B.3) Using something else

If you're not into virtualization solutions, you can setup your environment directly on your machine. Check the 
[software requirements list](./../getting-started/requirements).

<a name="Play"></a>
## C) Play

Now let's see it in action

1.a. Open your web browser and visit: 

- `http://hive.local` You should see an HTML page, with `HiveApi` in the middle.
- `http://admin.hive.local` You should see an HTML Login page.

1.b. Open your HTTP client and call: 

- `http://api.hive.local/` You should see a JSON response with message: `"Welcome to HiveApi."`, 
- `http://api.hive.local/v1` You should see a JSON response with message: `"Welcome to HiveApi (API V1)."`,   

2) Make some HTTP calls to the API:

> Heads up!
>
> To make HTTP calls you can use [Postman](https://www.getpostman.com/), [HTTPIE](https://github.com/jkbrzt/httpie) or 
any other tool you prefer.
>
> here is a postman file available that provides most of the pre-defined routes of HiveApi.

