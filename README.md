A personal introduction for Moodle on Red Had Openshift Express

To get started we clone Moodle into a directory and set up a remote tracking branch for stable releases (which is at 2.1 right now).

I also use a working directory called `~/openshift` where I store my openshift apps for easy maintenance:
	
	cd ~/openshift
	mkdir moodlestage
	cd moodlestage
    git clone git://git.moodle.org/moodle.git
	cd moodle
	git checkout --track origin/MOODLE_21_STABLE

Back in `~/openshift` I create a new app called moodle with my rhc client:

	rhc-create-app -a moodle -t php-5.3 -l yourlogin

The reason we stored moodle in `moodlestage` was to let the rhc client create a `moodle` directory for our app.

If everything was set up properly rhc will tell you the new address of your app and its remote git repository.

Now we need to set up MySQL so we can use it with Moodle:

	rhc-ctl-app -a moodle -e add-mysql-5.1 -l yourlogin

If successful it will return some information that you should save somewhere: username, password, db name and connection URL.

Now it's time to put moodle into its proper directory which we need to clean up first:

	rm –rf  ~/openshift/moodle/php/*
	
then we copy:
	
	cp –av ~/openshift/moodlestage/moodle/* ~/openshift/moodle/php

The next step will set up Moodle's config file so it can connect to the databse and do other great things. The following lines are somewhat compressed as for brevity I left out all the comments. Normally you would edit the `config-dist.php` file and then rename it to `config.php`. If you are lazy create that file and copy these lines into it (but make sure you use your domain name for wwwroot):

	<?PHP
	unset($CFG);
	global $CFG;
	$CFG = new stdClass();
	
	$CFG->dbtype    = 'mysqli';
	$CFG->dblibrary = 'native';
	$CFG->dbhost    = $_ENV['OPENSHIFT_DB_HOST'];
	$CFG->dbname    = $_ENV['OPENSHIFT_APP_NAME'];
	$CFG->dbuser    = $_ENV['OPENSHIFT_DB_USERNAME'];
	$CFG->dbpass    = $_ENV['OPENSHIFT_DB_PASSWORD'];
	$CFG->prefix    = 'mdl_';
	$CFG->dboptions = array(
	    'dbpersist' => false,
	    'dbsocket'  => false,
	    'dbport'    => $_ENV['OPENSHIFT_DB_PORT'],
	);
	$CFG->wwwroot   = 'https://moodle-yourdomain.rhcloud.com';
	$CFG->sslproxy=true;
	$CFG->dataroot  = $_ENV['OPENSHIFT_DATA_DIR'];
	$CFG->directorypermissions = 02777;
	$CFG->admin = 'admin';
	require_once(dirname(__FILE__) . '/lib/setup.php');

Note that we used an *https* URL for secure connections. We therefore had to add `$CFG->sslproxy=true;` to the config file.

Once this is finished we need to add all the new files and commit it to the new repository:

	cd ~/openshift/moodle
	git add .
	git commit –am "Initial commit for Moodle"

Now we can push to Openshift and be ready for installing Moodle:

	git push

You should now be able to visit your new app and start the install script.

*Have fun.*