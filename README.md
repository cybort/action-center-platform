[![Build Status](https://travis-ci.org/EFForg/action-center-platform.svg?branch=master)](https://travis-ci.org/EFForg/action-center-platform)


Action Center Platform
======================

The Action Center Platform is an online organizing tool maintained by [EFF](https://www.eff.org/). Administrators can create targeted campaigns where users sign petitions, contact legislators, and engage on social media.


## Setup

Follow these instructions to run the Action Center using Docker (recommended). To run the Action Center without Docker, see [setup without Docker](https://github.com/EFForg/action-center-platform/wiki/Setup-without-Docker).

1. Install Docker ([instructions](https://docs.docker.com/engine/installation/)) and Docker Compose ([instructions](https://docs.docker.com/compose/install/)).
2. `git clone https://github.com/EFForg/action-center-platform.git`
3. Copy `docker-compose.yml.example` to `docker-compose.yml`, and `.env.example` to `.env`. Fill in the variables in `.env` according to the instructions in that file. See [notable dependencies](#notable-dependencies) for hints.
4. Build the docker image: `sudo docker-compose build`
5. Run the application: `sudo docker-compose up`
6. In a new tab, get a bash shell with access to your app: `sudo docker-compose exec app bash`.
    1. If you aren't running migrations automatically, run `rake db:migrate` to migrate the database.
    2. Run `rake congress:update` to populate CongressMember table.


### Notable Dependencies

* Amazon S3 secret key and key id
  * Allows admins to upload images for the ActionPages
* [SmartyStreets API](https://smartystreets.com/account/create) key and id
  * Allows Congress members to be looked up for users
* [Phantom of the Capitol](https://github.com/efforg/phantom-of-the-capitol) whitelisting on server side?
  * Allows users to submit e-messages to congress
* [Call Congress](https://github.com/EFForg/call-congress) url and API key
  * Connects calls between citizens and their congress person using the Twilio API


### Generating the icon font

The EFF icon font is generated using [fontello.com](fontello.com) and via the [fontello-rails-converter](https://github.com/railslove/fontello_rails_converter).

To add a new icon to the fontset, read the **Update your existing fontello font** section of the [fontello_rails_converter readme](https://github.com/railslove/fontello_rails_converter#updating-your-existing-fontello-font).


## Using the Action Center

Action Center administrators can create four types of actions:
* **Call Action**
  * A user is connected to a political leader by phone, leaving a message or sometimes speaking to an aid.
* **Petition Action**
  * A user signs a petition, leaving an email address and sometimes location data.
  * Optionally, users can petition local institutions (like universities) and see signatures by institution.
* **Tweet Action**
  * A user is invited to join a tweet action using their twitter account.
* **Email Action**
  * A user e-mails a target *or*
  * A user submits comments to a congressperson via [Phantom of the Capitol](https://github.com/efforg/phantom-of-the-capitol).


### Administering Users

To get started using the Action Center, create a user and grant them admin privileges. Administrators can create, track, and manage campaigns.

**To create an admin user**:

1. If the user doesn't exist yet, create them through the web interface by following the `register` link in the top nav.
2. Run the rake task to grant them admin access (including square brackets):
```
rake users:add_admin[youremail@example.org]
```
New users will need to complete an e-mail confirmation in order to log in. Administrators can access admin features by clicking `admin` in the nav.

**To remove an admin user**:
```
rake users:remove_admin[youremail@example.org]
```

**To list all admin users**:
```
rake users:list_admins
```

### Delayed Jobs and Cron

Action Center uses [DelayedJob](https://github.com/collectiveidea/delayed_job) to perform certain tasks outside of a web request context. See that repository for information regarding how to run a delayed job worker. If you are deploying with Docker, our [docker-compose.yml.example](https://github.com/EFForg/action-center-platform/blob/master/docker-compose.yml.example) shows how to create a service which processes the job queue indefinitely.

You may also want to automate certain other tasks (such as `rake signatures:deduplicate` and `rake congress:update`) to run occasionally. For those deploying with Docker, docker-compose.yml.example illustrates how to create a service which runs these commands periodically using cron.

### Embedding Actions

Embedding actions is simple. Just include the following HTML on the page you want the action to be embedded:

    <script type="text/javascript" src="https://act.eff.org/action/embed"></script>
    <a class="action-center-widget" href="https://act.eff.org/action/shut-the-nsa-s-backdoor-to-the-internet">Take part in the action!</a>

The link href should point to the action page you wish to embed. You may add `?nosignup=1` to the URL to get newsletter signup fields omitted from the action.

If you want to get fancy, you can modify the embed code to include some of the following parameters, all of which are optional:

    <script type="text/javascript">
        var ac_embed = {};
        ac_embed.css = "https://example.com/hello.css"; // specify a css file url
        ac_embed.width = 500; // specify a width manually
        ac_embed.no_css = true; // remove all default styles
        ac_embed.css_content = "#some_elem"; // specify an element which itself contains some styles
        ac_embed.bioguide_ids = ["ID1", "ID2"...] // bioguide IDs of congress members to target
    </script>
    <script id="some_div" type="text/x-css-content">
        body{
            background-color: blue;
        }
    </script>
    <script type="text/javascript" src="https://act.eff.org/action/embed"></script>
    <a id="action-center-widget" href="https://act.eff.org/action/shut-the-nsa-s-backdoor-to-the-internet">Take part in the action!</a>


## Testing

To run the full test suite, simply run `rake` with no arguments.

Rspec tests are used for unit testing the app, and some integration testing. Cucumber tests are used for testing API keys, javascript tests, and feature tests.

## Linting

`rake` will also run our linting:

### [Rubocop](https://github.com/bbatsov/rubocop)
Rubocop checks for consistent style across the Ruby areas of the codebase.  We use a modified version of [Rubocop-Github](https://github.com/github/rubocop-github) to stay consistent with [SEC](https://github.com/EFForg/sec).

### [Sass Lint] (https://github.com/sasstools/sass-lint)
Sass-lint checks for consistent style across the stylesheets. Our
.sass-lint file is derived from [SEC](https://github.com/EFForg/sec).

## Acknowledgements

This project was created by Lilia Kai, Thomas Davis, and Sina Khanifar. Large portions of the codebase are directly attributable to them, while under the employ or contractorship of the Electronic Frontier Foundation in 2014. Thank you Lilia, Thomas, and Sina! The Action Center is currently maintained by the EFF Engineering and Design team.


## Licensing

See the `LICENSE` file for licensing information. This is applicable to the entire project, sans any 3rd party libraries that may be included.
