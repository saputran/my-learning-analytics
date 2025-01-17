# My Learning Analytics
[![Codacy Badge](https://api.codacy.com/project/badge/Grade/0fd487531e244c0ebbfbc25e8753c484)](https://app.codacy.com/app/ITS_Teaching_And_Learning/student-dashboard-django?utm_source=github.com&utm_medium=referral&utm_content=tl-its-umich-edu/student-dashboard-django&utm_campaign=Badge_Grade_Settings)

My Learning Analytics (MyLA) is a [learning analytics](https://en.wikipedia.org/wiki/Learning_analytics) platform designed for students to view their own learning data generated in the [Canvas Learning Management System](https://www.instructure.com/canvas/?newhome=canvas). It currently has 3 views ([Resources Accessed](https://sites.google.com/umich.edu/my-learning-analytics-help/home/resources-accessed), [Assignment Planning](https://sites.google.com/umich.edu/my-learning-analytics-help/home/assignment-planning), and [Grade Distribution](https://sites.google.com/umich.edu/my-learning-analytics-help/home/grade-distribution)), with more views planned in the future.

## Getting Started
These instructions will get a copy of MyLA up and running on your local machine with anonymized/fake student data.

### Prerequisites
1. **Install [Docker](https://www.docker.com/)**.
1. **Install [Git](https://git-scm.com/downloads)**.

### Installation and Setup
1. Clone this repo. `git clone https://github.com/tl-its-umich-edu/my-learning-analytics.git`
1. Then cd into the repo. `cd my-learning-analytics`
1. Create a directory in your home directory called "mylasecrets". `mkdir ~/mylasecrets`. This directory is mapped by docker-compose.yml into the container as /secrets/
1. Copy the config/env_sample.hjson into mylasecrets as env.hjson. `cp config/env_sample.hjson ~/mylasecrets/env.hjson`
1. Copy the config/cron.hjson file into mylasecrets. `cp config/cron.hjson ~/mylasecrets/cron.hjson`
1. Examine the `env.hjson` file, you may need to change some of the configuration in it now or later. There are comments to help the configuration.
1. Create a new `.env` file and copy the values from `.env.sample`, which has the suggested default environment variable settings.
1. Examine the `.env` file. It mostly just has the MySQL information as well as locations of the environment files.
1. Start the Docker build process (this will take some time). `docker-compose up -d --build`
1. Download the latest SQL file from this link: https://drive.google.com/drive/u/0/folders/1Pj7roNjRPGyumKKal8-h5E6ukUiXTDI9.
1. Load database with data. `docker exec -i student_dashboard_mysql mysql -u student_dashboard_user --password=student_dashboard_pw student_dashboard < {name of sql file}`

You may also optionally place the json settings directly into the `ENV_JSON` environment variable if your deployment environment doesn't easily support mounting the `env.hjson` file into container. When using `ENV_JSON` put the entire contents of `env.hjson` into it as single line string.

#### Logging in as admininstrator
1. Navigate to http://localhost:5001/ and log in as:
    ```
    username: root
    password: root
    ```
    
1. As you are now logged in as `root`, there are no courses listed. Next 3 steps will help you view a sample course.

1. Connect to MySQL database.
    ```
    Host: 127.0.0.1
    Username: student_dashboard_user
    Password: student_dashboard_pw
    Database: student_dashboard
    Port: 5306
    ```
    
1. Navigate to `course` table and select a `canvas_id` value, which will be used in the next step.

    ```mysql
    SELECT canvas_id FROM course
    ```

1. Go to http://localhost:5001/courses/***`canvas_id`*** with the `canvas_id` you found. (For example, with SQL file `myla_test_data_2019_10_16.sql` loaded, Go to http://localhost:5001/courses/235420 or http://localhost:5001/courses/362855 and view the course page as an admin.) 

1. To get to the Django admin panel, click the avatar in the top right, then click `Admin`, or go here: http://localhost:5001/admin.

#### Logging in as a student
1. Click on the top-right circle, then click `Logout`.

1. Connect to MySQL database.
    ```
    Host: 127.0.0.1
    Username: student_dashboard_user
    Password: student_dashboard_pw
    Database: student_dashboard
    Port: 5306
    ```
    
1. Pick a student `sis_name` from the`user` table to be used in the next step.

    ```sql
    SELECT sis_name FROM `user` WHERE enrollment_type = 'StudentEnrollment'
    ```

    

1. Create an authorized user.
    ```sh
    docker exec -it student_dashboard python manage.py createuser --username={insert sis_name} --password={create password} --email=test@test.com
    ```
    
    Note: To make a user a superuser, edit the record in `auth_user` to set `is_staff=1` and `is_superuser=1`.
    
1. Login using the username and password created.

1. The course(s) enrolled by the student with selected `sis_name` will be displayed. Click on a course to view as the student selected in step 3.

## MyLA Configuring Settings
- If you were using a prior version of MyLA, there is a utility `env_to_json.py` to help convert your configuration. Running `python env_to_json.py > config/env.hjson` should create your new config file from your `.env`file.

### Secrets (Optional)
The `bq_cred.json` defined in the `.env` file is service account for BigQuery, and it needs to be supplied and put into the directory defined in the `.env` file and setup in the environment.

### Control course view options
View options can be controlled at the global and course level. If a view is disabled globally, it will be disabled for every course, even if previously enabled at the course level. If a view is not globally disabled, it can still be disabled at the course level.

`VIEWS_DISABLED` is the comma delimited list of views to disable (default empty). The expected name of the view is the same as the view's column name in the `course_view_option` table. For example `VIEWS_DISABLED=show_resources_accessed,show_grade_distribution` will disable both the Resources Accessed and Grade Distribution views.

'show_grade_type' is a field in the 'course' table that can either be 'Percent' or 'Point'. This specifies how the grades should be displayed in the different views in MyLA.  

Note that by default all views are enabled when a course is added.

### Control the primary user interface color
MyLA allows you to configure the primary color of the user interface (i.e., the color used for the top toolbar, or DashboardAppBar, and other structural components of the application). You can set this color by defining a key-value pair in `env.hjson`, with the key being `"PRIMARY_UI_COLOR"` and the value being a valid hex value. See `env_sample.hjson` for an example.

Other colors from the user interface are currently fixed and included in version control, as they are intentional design choices. You can see where color values are set in `assets/src/defaultPalette.js`.

### LTI v1.3 Configuration
* MyLA Supports LTI 1.3, using [pylti1.3](https://github.com/dmitry-viskov/pylti1.3) 
* Instructions for LTI 1.3 configuration are in the [MyLA wiki](https://github.com/tl-its-umich-edu/my-learning-analytics/wiki/Dev%7CDeploy:-Configuration-for-LTI-1.3)

* You also need to configure CSP value in the environment, specifically the `FRAME_SRC.` (See next section) In addition you need to ensure you are using https and `CSRF_COOKIE_SECURE` is true with your domain (or instructure.com) in trusted origins.

* You may optionally disable Deployment Id validation by settings `LTI_CONFIG_DISABLE_DEPLOYMENT_ID_VALIDATION` to `True` (default `False`).

### Content Security Policy

All of the Content Security Policy headers can be configured. In the `env_sample.hjson` there is a sample security policy that should work to bring it up. It has `REPORT_ONLY` set to true; by default it won't actually do anything. If you're using LTI to embed this tool or you want to configure the policy you need to adjust these values and set `REPORT_ONLY` to false.

### Populate initial terms and courses using demo
A `demo_init.sh.sample` file has been provided to help initialize terms and courses. This can be used in combination
with `cron.py` to provide some data to start exploring the tool's features. Rename the file to `demo_init.sh` and
replace the provided fabricated values with those of terms and courses relevant to your local institution, providing
additional courses or terms as desired. Ensure that the `CANVAS_DATA_ID_INCREMENT` environment variable is set
appropriately in `dashboard/settings.py`, and then run the following command:


    docker exec -it student_dashboard /bin/bash ./demo_init.sh


If you have problems, you can connect direct into a specific container with the command

    docker-compose run web /bin/bash

### Openshift process
You should login via Shibboleth into the application. Once you do that for the first admin you'll have to go into the database `auth_user` table and change `is_staff` and `is_superuser` to both be true. After doing this you can change future users with any admin via the GUI.

### Load user, file, file access data into database
Users and files are loaded now with the cron job. This is run on a separate pod in Openshift when the environment variable `IS_CRON_POD=true`.

Crons are configured in this project with django-cron. Django-cron is executed whenever `python manage.py runcrons` is run but it is limited via a few environment variables.

The installation notes recommends that you have a Unix crontab scheduled to run every 5 minutes to run this command. https://django-cron.readthedocs.io/en/latest/installation.html

This is configured with these values
***(Django Cron) Run only at 2AM***

RUN_AT_TIMES=2:00

***(Unix Cron) - Run every 5 minutes***

CRONTAB_SCHEDULE=*/5 * * * *

For local testing, make sure your secrets are added and your VPN is active. Then run this command on a running container to execute the cronjob

`docker exec -it student_dashboard /bin/bash -c "python manage.py migrate django_cron && python manage.py runcrons --force"`

After about 30-60 seconds the crons should have completed and you should have data! In the admin interface there is a table where you can check the status of the cron job runs.

## Additional Resources
[Video guide](https://www.youtube.com/watch?v=CSQmQtLe594&feature=youtu.be) to setting up courses in the My Learning Analytics admin tool.

## Populating Copyright information in footer
1. Since MyLA can be used by multiple institution, copyright information needs to be entitled to institutions needs.
2. Django Flatpages serves the purpose. The display of the copyright content can be controlled from the Django Admin view.
3. The url for configuring copyright info must be `/copyright/` since that is used in the `base.html` for pulling the info. [Read more here](https://simpleisbetterthancomplex.com/tutorial/2016/10/04/how-to-use-django-flatpages-app.html)

## Testing

The application currently uses two frameworks for front-end testing: [Cypress](https://www.cypress.io/) and [Jest](https://jestjs.io/). There are plans to implement back-end tests in the future.

### Cypress Testing

Some front-end tests are implemented using the [Cypress framework](https://www.cypress.io/). 

 For running cypress tests locally, it is essential that you have Myla instance running locally. Launch Myla from the
 browser go to the admin view and add user called `donald07` with password `root`, first name `donald`, and last name `07`. Get the latest depersonalized datadump
 as described from the Step 9 in [installation and setup](#installation-and-setup).

 Install cypress 

 `npm install cypress`

 and install the plugins add-on with

 `npm i cypress-plugin-snapshots -S`

 Cypress can be started with the command

 `npm run cypress:open`

 When running tests do not use the All Tests button due to unsolved issues. 
 Run the cypress test if UI change are there as part of the work.
 If a snapshot fails due to change in the UI, try updating the snapshot from the failed test from cypress controlled 
 browser 'compare snapshot' and a pop up appears to `Update Snapshot` 

## Jest Testing
Other front-end tests leverage the [Jest framework](https://jestjs.io/). 

To run the Jest test suite, execute the command `docker exec -it webpack_watcher npm test`

To update snapshots, execute `docker exec -it webpack_watcher npm run-script update-snapshot`

## Data Validation Testing

Data validation scripts are in scripts/data_validation folder. To run the data validation scripts, follow the steps below:

1. Copy env.hjson file from env_sample.hjson file
   ```sh
   cp scripts/data_validation/env_sample.hjson scripts/data_validation/env.hjson
   ```
2. Update the configuration values. Note the hard-coded Michigan specific filters in various query strings. Please adjust those values for your institution usage.

3. Run data validation script within docker container:

   - Validate queries using UDP BigQuery Expanded Table: This script will validate queries against the UDP BigQuery `expanded` table, and compare whether the data results are identical with those returned from the `events` table queries.

     ```sh
     docker exec -it student_dashboard /bin/bash -c "python scripts/data_validation/validate_udp_events_vs_expanded.py"
     ```
    - Validate UDP context store queries: This script will validate queries against the UDP context_store tables, and compare whether the data results are identical with those returned from Unizin Data Warehouse(UDW) tables.
        - Before running this queries, please make sure there are two cron queries files in ~/mylasecrets folder, naming cron.hjson (for UDW queries ) and cron_udp.hjson (for UDP queries) 

     ```sh
     docker exec -it student_dashboard /bin/bash -c "python scripts/data_validation/validate_udw_vs_udp.py"
     ```
## Accessibility

### Keyboard Navigation
Users that use Safari or Firefox on macOS that would like to enable keyboard navigation in MyLA will have to follow the steps [here](https://github.com/tl-its-umich-edu/my-learning-analytics/wiki/Accessibility).

## Contributing to MyLA
* [Contribution Guide](CONTRIBUTING.md)

## License check
MyLA is licenced under Apache v2.0. There is a file `myla_licence_compat.ini` that can be used with [Python Licence Check](https://github.com/dhatim/python-license-check) to check any new dependencies and their licences.