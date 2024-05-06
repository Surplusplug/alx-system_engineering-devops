# Postmortem Report

Following the release of ALX's System Engineering & DevOps project 0x19, at approximately 06:00 West African Time (WAT) in Nigeria, an outage was experienced on an isolated Ubuntu 14.04 container running an Apache web server. GET requests on the server resulted in 500 Internal Server Errors instead of the expected response, which should have been an HTML file defining a simple Holberton WordPress site.

## Debugging Process

Upon encountering the issue at around 19:20 PST, the debugging process was initiated.

1. Inspected running processes using `ps aux`, confirming two `apache2` processes - `root` and `www-data` - were operational.

2. Investigated the `sites-available` folder in `/etc/apache2/`, confirming that the web server was serving content from `/var/www/html/`.

3. Ran `strace` on the PID of the `root` Apache process in one terminal and curled the server in another, but obtained no useful information from `strace`.

4. Repeated step 3, this time with the `www-data` process PID, which revealed an `-1 ENOENT (No such file or directory)` error when attempting to access `/var/www/html/wp-includes/class-wp-locale.phpp`.

5. Methodically searched through files in `/var/www/html/` using Vim pattern matching, identifying the erroneous `.phpp` file extension in `wp-settings.php` (Line 137).

6. Corrected the typo by removing the trailing `p` from the line.

7. Conducted another `curl` test on the server, which returned a successful 200 status.

8. Developed a Puppet manifest to automate the resolution of such errors.

## Summary

In essence, the outage resulted from a simple typo. Specifically, the WordPress application encountered a critical error in `wp-settings.php` when attempting to load `class-wp-locale.phpp`, whereas the correct filename, located in the `wp-content` directory, should have been `class-wp-locale.php`. The fix involved removing the extraneous `p` from the filename.

## Preventive Measures

To avert similar outages in the future, it's essential to:

* Prioritize testing of applications before deployment to catch errors early.
* Implement status monitoring systems like [UptimeRobot](https://uptimerobot.com/) to instantly alert of website outages.
