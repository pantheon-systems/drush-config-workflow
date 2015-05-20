# Installation

This document describes the components you will need in order to use the `drush config-merge` command.

You can quickly get started by running the `quickstart` script:
```
./scripts/quickstart [pantheon-site-name]
```
This will install all of the software that you need, and create a pair of local Drupal sites to work with.  Optionally, if you provide the name of a Drupal 8 Pantheon site as an argument to quickstart, then it will create a local copy of the "dev" environment for that site.

If you do not want to use the script, you can manually install the necessary components per the instructions below.

## Prerequirements

Insure that PHP 5.4 or later, and an SQL database are installed before you begin.

## Requirements

* A config-merge tool, e.g. `kdiff3`, must be installed
* Drush 7.0.0-rc2 (only), or Drush 8.x-dev
* Pantheon Terminus CLI (Optional -- only needed if using a Pantheon remote)
* Drupal 8.0-beta10 or later
  * A local or remote "testing" or "staging" site
  * A local Drupal "working" site

## Installing kdiff3

* **Mac:** brew install Caskroom/cask/kdiff3
* **Linux:** sudo apt-get install kdiff3
* **Windows:** When using Windows, it is recommended to use Drush and Drupal inside of a virtualized environment, such as [kalabox](http://www.kalamuna.com/products/kalabox/) or [Varying Drupal Vagrants](https://github.com/gman29/varying-drupal-vagrants/blob/master/www/vvv-hosts).

## Installing Drush 7 or 8
Follow the [installation instructions in the Drush documentation](http://docs.drush.org/en/master/install/) to install Drush 7.  Most users should be able to upgrade from Drush 5 or Drush 6 to version 7 without encountering any issues.  At the time of this writing, Drush 7.0.0-rc2 is recommended.  However, once Drush 8.0-alpha1 is released, this version should be used with Drupal 8, as at the moment, only this version of Drupal is supported by the config-merge command.  A Drupal 7 version may be provided in the future.

## Installing Terminus CLA

Follow the [terminus installation instructions](https://github.com/pantheon-systems/cli/wiki/Installation).

## Installing Drupal 8
If you already have a remote Drupal 8 testing site, then you may use it to try out config-merge.  Otherwise, you'll need to make a pair of local sites to work with.  These sites should share the same git repository, and they should have the same site UUID--meaning, that the database of the working site should be a copy of the databse on the first site.

You can create the first site on Pantheon, or create it locally using `drush qd`:

* **Remote, on Pantheon:** [Spin up a remote Drupal 8 environment](https://dashboard.pantheon.io/products/drupal8/spinup)
* **Local, with Drush:** `drush qd sitename --core=drupal-8.0.x --db-url='mysql://root@localhost:/dbname'`

Adjust your database credentials as necessary, if using Drush to create a local site.

Once you have made the first site, you will need to clone it.

### Make a Local Clone of a Pantheon Site

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
$aliases["pantheon-site-name"] = array(
  'root' => '/Users/username/local/drupal/pantheon-site-name',
  'uri' => 'localhost:8778',
);
```
Change `pantheon-site-name` to match the name of your site, and adjust the path to the site and the port number in the URI to suit your preferences.

3. Check out the code for the Pantheon site.

Look up your repository's URL on the Pantheon dashboard, or use Terminus to look up the site's UUID:
```
$ UUID=$(terminus site info --site=pantheon-site-name --field=id)
```
Make sure that the parent directory exists (i.e., `mkdir /Users/username/local/drupal`), then `cd` to it and clone the repository.
```
$ git clone ssh://codeserver.dev.$UUID@codeserver.dev.$UUID.drush.in:2222/~/repository.git pantheon-site-name
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
$config['system.site']['name'] = 'Local pantheon-site-name';

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
$ drush @local.pantheon-site-name sql-create
$ drush sql-sync @pantheon.pantheon-site-name.dev @local.pantheon-site-name
```

Make sure that the alias name you use for the target site matches what you did in step 2, above.

Once you have completed all of these steps, you will be ready to start using the config-merge command, as described in the (README)[README.md].

### Make a Local Clone of Another Local Site

tbd
