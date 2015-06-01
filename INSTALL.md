# Installation

This document describes the components you will need in order to use the `drush config-merge` command.

You can quickly get started by running the `quickstart` script:
```
./bin/quickstart [--create-local-sites name] [pantheon-sitename]
```
This will install all of the software that you need, and create a local development copy of a Drupal 8 Pantheon site (if the Pantheon site name is provided as a commandline argument), or a pair of local Drupal 8 sites (if --create-local-sites is used).

If you do not want to use the script, you can manually install the necessary components per the instructions below.

## Prerequirements

Insure that PHP 5.4 or later, and MYSQL are installed before you begin.

## Requirements

* A config-merge tool, e.g. `kdiff3`, must be installed
* Drush 8.x-dev
* Pantheon Terminus CLI (Optional -- only needed if using a Pantheon remote)
* Drupal 8.0-beta10 or later
  * A local or remote "testing" or "staging" site
  * A local Drupal "working" site

## Installing kdiff3

* **Mac:** brew install Caskroom/cask/kdiff3
* **Linux:** sudo apt-get install kdiff3
* **Windows:** When using Windows, it is recommended to use Drush and Drupal inside of a virtualized environment, such as [kalabox](http://www.kalamuna.com/products/kalabox/) or [Varying Drupal Vagrants](https://github.com/gman29/varying-drupal-vagrants/blob/master/www/vvv-hosts).

## Installing Drush 8
Follow the [installation instructions in the Drush documentation](http://docs.drush.org/en/master/install/) to install Drush 8.  Most users should be able to upgrade from Drush 5 or Drush 6 to version 8 without encountering any issues.  Currently, only Drupal 8 is supported by the config-merge command.  A Drupal 7 version may be provided in the future.

## Installing Terminus CLA

Follow the [terminus installation instructions](https://github.com/pantheon-systems/cli/wiki/Installation).

## Installing Drupal 8
If you already have a remote Drupal 8 testing site, then you may use it to try out config-merge.  Otherwise, you'll need to make a pair of local sites to work with.  These sites should share the same git repository, and they should have the same site UUID--meaning, that the database of the working site should be a copy of the databse on the first site.

You can create the first site on Pantheon, or create it locally using `drush qd`:

* **Remote, on Pantheon:** [Spin up a remote Drupal 8 environment](https://dashboard.pantheon.io/products/drupal8/spinup)
* **Local, with Drush:** `drush qd sitename --core=drupal-8.0.x --db-url='mysql://root@localhost:/dbname'`

Adjust your database credentials as necessary, if using Drush to create a local site.

Once you have made the first site, you will need to clone it.

### Make a Local Clone of Another Local Site

If you created a Drupal 8 site with `drush quick-drupal`, follow the instuctions in this section.  If you are using a Pantheon site, skip ahead to the next section, _Make a Local Clone of a Pantheon Site_, below.

Reminder:  You can just run `./bin/quickstart --create-local-sites sitename`, and it will do all of the steps described below for you.

1. Make an alias for both of your sites.

Create a file ~/.drush/local.aliases.drushrc.php, and add the following entry:
```
$aliases["sitename.dev"] = array(
  'root' => '/Users/username/local/drupal/pantheon-sitename',
  'uri' => 'localhost:9966',
);
$aliases["sitename.test"] = array(
  'root' => '/Users/username/local/drupal/pantheon-sitename',
  'uri' => 'localhost:9999',
);
```
Change `sitename.dev` and `sitename.test` to match the name of your site, and adjust the path to the site and the port number in the URI to suit your preferences.

2. Decide where you want to put your configuration directories:

Drupal 8 defaults to storing configuration exports inside the `files` directory.  If you use the example.gitignore file that Drupal 8 also provides, then the `files` directory is not committed to the git repository.  We definitely want our configuration to be under git version control, so that we can use git's three-way-merge facilities.

Add something like the following to your settings.php file:
```
\$config_directories['active'] = 'sites/default/files/config_RANDOMSTRING/active';
\$config_directories['staging'] = 'sites/default/config_RANDOMSTRING/staging';
```

3. Set up your .gitignore file

Copy Drupal's example.gitignore file to .gitignore, and commit it to the repository.

If you do not want to change the location of your configuration files as shown in the previous step, you could instead adjust the contents of your .gitignore file to include the configuration directory in the repository.

4. Set up a git repository for the test site

At the root of your test site, set up a repository:
```
git init
git add .
git commit -m "Initial commit."

5. Set up a central repository

If you do not already have a central repository available on a server, you can set one up on your local machine, just to use for testing `config-merge` in a simulated environment.
```
mkdir -p /path/to/git/repo
cd /path/to/git/repo
git init --bare
cd /path/to/test/drupal/site
git remote add origin /path/to/git/repo
git push origin master
```

6. Clone the "test" Drupal site to a "dev" Drupal site

git clone /path/to/git/repo /path/to/dev/drupal/site

7. Configure your "dev" site's settings.php file

Edit the "Databases" record in your dev site's .settings.php file and give the dev site its own local database.  Make sure that you create the sql database (e.g. via `drush sql-create`).

You might also wish to set the site name in settings so that you can tell the difference between the two sites:
```
$config['system.site']['name'] = 'Local sitename'
```

8. Copy the "test" site's dateabase to the "dev" site database
```
drush sql-sync @sitename.test @sitename.dev
```

### Make a Local Clone of a Pantheon Site

If you created a Drupal 8 site on Pantheon, follow the instructions in this section.

Reminder:  You can just run `./bin/quickstart pantheon-sitename`, and it will do all of the steps described below for you.

1. Make sure that your Pantheon Aliases are up-to-date:

Use Terminus to download a copy of your aliases.
```
$ terminus sites aliases
```
This will create the file ~/.drush/pantheon.aliases.drushrc.php; you can override this with the --location option, though.

n.b. You will need to log in via `terminus auth login` first.

2. Make an alias for your local site.

Create a file ~/.drush/local.aliases.drushrc.php, and add the following entry:
```
$aliases["pantheon-sitename"] = array(
  'root' => '/Users/username/local/drupal/pantheon-sitename',
  'uri' => 'localhost:8778',
);
```
Change `pantheon-sitename` to match the name of your site, and adjust the path to the site and the port number in the URI to suit your preferences.

3. Check out the code for the Pantheon site.

Look up your repository's URL on the Pantheon dashboard, or use Terminus to look up the site's UUID:
```
$ UUID=$(terminus site info --site=pantheon-sitename --field=id)
```
Make sure that the parent directory exists (i.e., `mkdir /Users/username/local/drupal`), then `cd` to it and clone the repository.
```
$ git clone ssh://codeserver.dev.$UUID@codeserver.dev.$UUID.drush.in:2222/~/repository.git pantheon-sitename
```

4. Set up your .gitignore file

If you haven't already done so, copy Drupal's example.gitignore file to .gitignore, and commit it to the repository.

5. Configure your settings.php file

Add a line at the bottom of your settings.php file, and commit it:

```
if (file_exists(__DIR__ . "/settings.local.php")) {
  include __DIR__ . "/settings.local.php";
}
```

6. Define local configuration values in your settings.local.php file

Add local configuration similar to the example shown below, customized for your environment:
```
$config['system.site']['name'] = 'Local pantheon-sitename';

$settings['hash_salt'] = 'RMiVntIAbUxQYG8SbO1DkXLEYTNZdGzsDRF3-axZ_WRi96rqOskbitY7byMDkUx4QUIMZtvfWQ';

$databases['default']['default'] = array (
  'database' => 'pantheonsitenamedb',
  'username' => 'root',
  'password' => '',
  'prefix' => '',
  'host' => 'localhost',
  'port' => '',
  'namespace' => 'Drupal\\Core\\Database\\Driver\\mysql',
  'driver' => 'mysql',
);
```
The most important items to customize are the Database username, and the database password, which must match the credentials for a local user that has permissions to create and write to local databases.

The value of the hash salt is not very important if the site will only be used from the local system.  You can generate a random 75-character hash salt from bash via the following command:
```
$ cat /dev/urandom | tr -dc 'a-zA-Z0-9' | fold -w 75 | head -n 1
```

7. Copy the database from Pantheon to the local site

```
$ drush @local.pantheon-sitename sql-create
$ drush sql-sync @pantheon.pantheon-sitename.dev @local.pantheon-sitename
```

Make sure that the alias name you use for the target site matches what you did in step 2, above.

Once you have completed all of these steps, you will be ready to start using the config-merge command, as described in the (README)[README.md].

8. Export the configuration files if they have never been exporte before

Run `drush config-export`, and commit the results back to the git repository.

## Configuration merge experimentation

Once you have two Drupal 8 sites set up, you can use them for `config-merge` experimentation.  If you cloned a Pantheon site locally, you must either set up a local webserver to serve the site, or just run it using the Built-in webserver provided by Drush:
```
drush rs @local.sitename /
```
If you set up two separate local sites, then you'll need to run both of them. Enter the commands below in two separate terminal windows (one line in each terminal) to run both sites using the PHP built-in webserver.
```
drush rs @local.sitename.test
drush rs @local.sitename.dev
```
As an experiment, try changing the site slogan on one site, and move the search block to the header region on the other site.  You can then use `drush @local.sitename.dev config-merge @local.sitename.test` (if you set up two local sites) or `drush @local.sitename config-merge @pantheon.sitename.dev` (if you cloned a Pantheon site).  When config-merge completes, you should find that the local site has both the new site slogan, and the search block should be in the header.  The test site won't get the updated configuration changes until you deploy them.

To simulate a "deploy" on the two local sites, run:
```
cd /path/to/dev
git push
cd /path/to/test
git pull
drush @local.sitename.test config-import
```
To deploy from the local site to the Pantheon site it was cloned from, first commit your configuration changes to git (config-merge will do this step), push it to the central repository (git push origin master), and then import the configuration changes on the Pantheon site with:"
```
drush @pantheon.sitename.dev config-import
```
Be aware that deploying like this will overwrite all of the configuration on the target site, so be sure to run config-merge first, so that you can insure that all of the remote configuration changes will be preserved.
