# Crumby

Crumby is an open source application for tracking and reporting visitor usage of
a website. Crumby is a Flask application so it works with well known tools such
as Apache, MySQL, and Python. Crumby includes a simple web-based frontend and a
JSON compliant RESTful API for reporting. Queries are assigned to a public or
private endpoint. Private queries are only accessible by authenticated users.

## Architecture

![architecture analytics][arch_analytics]

### Crumby App

The Crumby app is developed with Python and Flask so it can be used with any
server that supports the Web Server Gateway Interface (WSGI). The Crumby app
includes a command line interface for administration, a reporting module, and
a tracking module.

### Crumby CLI

The command line interface is used to administer the Crumby app, try `crumby -h`
from the terminal to print help on this command.

    usage: crumby [-h] {adduser,deluser,env,geoip,init,run,users} ...

    A command line interface for the Crumby web analytics app.

    positional arguments:
      {adduser,deluser,env,geoip,init,run,users}
        adduser             add user - grant private query access
        deluser             delete user - revoke private query access
        env                 display Flask configuration values
        geoip               download latest geoip database
        init                create template config file
        run                 launch Crumby in development server
        users               view list of users

    optional arguments:
      -h, --help            show this help message and exit

### Reporting Module

Apart from querying the database directly, visit and event records can be viewed
at a webpage or consumed through the JSON compliant RESTful API. The default
locations for these resources are:

    Webpage: https://<crumby.host>
    API: https://<crumby.host>/api

Crumby ships with some built in queries which show common tracking metrics (e.g.
visits per day, most popular pages, visitors location of origin). These queries
are segmented into two groups: public and private. Public queries can be
accessed by anyone. Private queries require authentication.

### Tracking Module

The tracking module contains the cmb.js script and configures endpoints for
receiving tracking data.

### Crumby Database

A SQL database is required to store tracking data. Any database compatible with
SQLAlchemy can be used. The following tables will be initialized when Crumby
first runs:

 * calendar - A range of dates available for queries
 * events - Visitor interactions with the website
 * users - Users permitted to access private queries
 * visits - Visitors to the website

#### calendar

| Column    | Category | Description                  |
|-----------|----------|------------------------------|
| datetime  | General  | Date and time                |


#### events

| Column    | Category | Description                  |
|-----------|----------|------------------------------|
| id        | General  | Unique record ID             |
| ip        | General  | IP address                   |
| cid       | General  | Visitor ID (cookie)          |
| datetime  | General  | Date and time event occurred |
| doc_title | Page     | Page title                   |
| doc_uri   | Page     | Page URI                     |
| name      | Action   | Name of the event            |
| value     | Action   | Value of the event           |

#### users

| Column   | Category | Description      |
|----------|----------|------------------|
| id       | General  | Unique record ID |
| username | Login    | Username         |
| password | Login    | Hashed password  |

#### visits

| Column          | Category          | Description                  |
|-----------------|-------------------|------------------------------|
| id              | General           | Unique record ID             |
| ip              | General           | IP address                   |
| cid             | General           | Visitor ID (cookie)          |
| datetime        | General           | Date and time page accessed  |
| doc_title       | Page              | Page title                   |
| doc_uri         | Page              | Page URI                     |
| doc_enc         | Page              | Encoding of page             |
| referrer        | General           | Referrer from client browser |
| \_referrer      | General           | Referrer from request header |
| platform        | Hardware/Software | Browser platform             |
| browser         | Hardware/Software | Browser name                 |
| version         | Hardware/Software | Browser version              |
| screen_res      | Hardware/Software | Browser screen resolution    |
| screen_depth    | Hardware/Software | Browser screen depth         |
| continent       | Geospatial        | Continent                    |
| country         | Geospatial        | Country                      |
| subdivision_1   | Geospatial        | Subdivisions                 |
| subdivision_2   | Geospatial        | Subdivisions                 |
| city            | Geospatial        | City                         |
| latitude        | Geospatial        | Latitude                     |
| longitude       | Geospatial        | Longitude                    |
| accuracy_radius | Geospatial        | Geo accuracy                 |
| time_zone       | Geospatial        | Time zone                    |
| lang            | General           | Language from client browser |
| \_lang          | General           | Language from request header |

### Geospatial Database (geoip.mmdb)

Crumby geolocates visits with the MaxMind geoip2 library and the
[GeoLite2 City](https://dev.maxmind.com/geoip/geoip2/geolite2/) database.

![architecture diagram][arch_web]

### Web Pages

Tracking visits to a webpage with Crumby is simple. Just add a pointer to the
analytics server cmb.js script. The pointer should be included in the source
code of any page that should be tracked, likely at the bottom of the `body` to
allow other content to load first.

    <script src="https://<crumby.host>/cmb.js"></script>

To track events (e.g. button click), make a call to `cmb.event(<name>, <value>)`
when the event is triggered.

## Getting Started

1. Setup WSGI Hosting Environment:

  There are many ways to deploy flask applications like crumby. Refer to the
  [Flask Deployment Options][flask_deployment] for several Hosted and
  Self-hosted options. At a minimum a [WSGI server][wsgi_server] environment is
  needed to deploy the crumby app.

2. Install a SQL Database:

  * Install a database supported by [SQL Alchemy][sql_alchemy]
  * Install associated python database drivers (e.g. pymysql)

3. Install Crumby:

  Installation will vary depending on the WSGI hosting environment selected.

  For hosted options, you may need the package source code, which can be found
  on github:

        git clone https://github.com/bmweiner/crumby.git

  For self-hosted options, you may want to install Crumby from PyPI:

        pip install crumby

  > Note: Some Crumby dependencies require extra software to build properly. If
  > you encounter errors during install, try installing gcc, libffi, and
  > python development headers, before installing Crumby.
  > **Hint:** sudo yum install gcc libffi-devel python-devel

4. Install the latest geoip database:

        cd ~
        crumby geoip

5. Initialize an example config file and enter configuration values:

        crumby init
        vi crumby.cfg

  See [Crumby Configuration](#crumby-configuration) for a description of available
  configuration options. Current configuration variables values can be viewed
  with the crumby CLI:

        crumby env

6. Tell crumby how to find the configuration file by setting an environment
   variable named: CRUMBY_SETTINGS to the config file path:

        export CRUMBY_SETTINGS=/Users/username/crumby.cfg

7. Deploy Crumby:

  Follow instructions for the [deployment option][flask_deployment] selected
  to deploy the crumby app.

## Crumby Configuration

Crumby uses a configuration file to setup the application environment.
Configuration values are used by Crumby and it's dependencies at runtime. In
addition to the following commonly used configuration values, please refer to
[Flask][flask_config], [Flask-SQLAlchemy][flask_sqlalchemy_config], and
[Flask-Login][flask_login_config] documentation for their respective options.

Set the value of an environment variable named `CRUMBY_SETTINGS` to the location
of the crumby configuration file and restart your server to apply changes. For
example:

    export CRUMBY_SETTINGS=/Users/username/crumby.cfg

| Key                            | Type               | Default                 | Value                                                                 |
|--------------------------------|--------------------|-------------------------|-----------------------------------------------------------------------|
| DOMAIN                         | str                | 'localhost:5000'        | Domain name of the server where crumby is installed.                  |
| SQLALCHEMY_DATABASE_URI        | str                | ‘sqlite:///~/crumby.db’ | SQL Database URL, follows the SQLAlchemy syntax.                      |
| GEOIP2_DATABASE_NAME           | str                | ‘~/GeoLite2-City.mmdb’  | Absolute path to the GeoIP database.                                  |
| SECRET_KEY                     | str                | None                    | Used to encrypt things. Must be set for SSL/TLS.                      |
| SESSION_COOKIE_SECURE          | bool               | False                   | Controls if the cookie should be set with the secure flag.            |
| PROXY_COUNT                    | int                | 0                       | Count of proxy server hops to remove when finding clients IP address. |
| CROSSDOMAIN_ORIGIN             | str or<br>iterable | ‘*’                     | URL(s) permitted to access the crumby API.                            |
| SQLALCHEMY_TRACK_MODIFICATIONS | bool               | False                   | Track modifications of objects and emit signals.                      |
| DEBUG                          |                    | False                   | enable/disable Flask debug mode                                       |
| SQLALCHEMY_ECHO                |                    | False                   | Log all the statements issued to stderr.                              |


## Development Server

Crumby can be run in the Flask development server (not recommended for
production) with the crumby CLI:

    crumby run

## Adding Queries

Crumby ships with a bunch of SQL queries which live in `Crumby/templates/api`
under the `private` or `public` directory. Public queries can be viewed by anyone
and private queries require authentication.

New queries can be added to the `public` and `private` directories. These
queries will be automatically added to the crumby webpage and API. The following
guidelines should be followed:

  * One text file per query with the extension `.sql`
  * The filename will be the name of the query
  * Queries must follow the syntax of the SQL database used

Refer to the [Crumby Database](#crumby-database) section for table names, field
names, and descriptions.

### Advanced Queries

Queries are templates processed by Flask using the Jinja2 syntax. Refer to the
Jinja2 [Template Designer Documentation][jinja2_template] to customize your
queries.

By default, every query is passed the following variables:

  * t0: Start Time, default today - 30 days
  * t1: End Time, default: today

These variables are created using query string parameters passed with a request
to the crumby API. Refer to the crumby API `https://<crumby.host>/api/public`,
for specifications on the API interface.

  * days, int. number of consecutive days to retrieve before today.
  * from, str in form YYY-MM-DD. start date of date range.
  * to, str in form YYY-MM-DD. end date of date range.

This is helpful if you want to constrain a query by time, for example returning
the count of unique users within a certain date range:

    SELECT date(datetime) as date, count(distinct cid) as users
    FROM visits
    GROUP BY date
    WHERE date(datetime) between date("{{t0}}") and date("{{t1}}")

## Managing Users

To view private queries, users must be authenticated. Users can be listed,
added, or removed with the crumby CLI:

    crumby users
    crumby adduser name password
    crumby deluser name

## Deployment Examples

Check out the `crumby\deployment` directory for some example deployment scripts
and configurations.

[wsgi_server]: http://wsgi.readthedocs.io/en/latest/servers.html
[sql_alchemy]: http://docs.sqlalchemy.org/en/latest/dialects/index.html
[flask_deployment]: http://flask.pocoo.org/docs/0.12/deploying/
[flask_config]: http://flask.pocoo.org/docs/0.12/config/
[flask_sqlalchemy_config]: http://flask-sqlalchemy.pocoo.org/2.1/config/
[flask_login_config]: https://flask-login.readthedocs.io/en/latest/
[jinja2_template]: http://jinja.pocoo.org/docs/2.9/templates/#template-designer-documentation

[arch_web]: https://raw.githubusercontent.com/bmweiner/Crumby/master/assets/arch_web.png
[arch_analytics]: https://raw.githubusercontent.com/bmweiner/Crumby/master/assets/arch_analytics.png
