//$Id$

WRAPlogin module v6.x-1.8
=========================
By brabec at ncsu.edu
Modified from Pubcookie by jvandyk at iastate.edu

DESCRIPTION
-----------
This module integrates WRAP-based authentication with Drupal.

For more information about WRAP, see http://webauth.ncsu.edu/wrap/

PREREQUISITES
-------------
Before using this module, make sure that your WRAP environment
is set up and running correctly.

The best way to do this is to create a subdirectory on your webserver
with an HTML file and an .htaccess file in it. The .htaccess file
would look something like this:

AuthType WRAP
require affiliation ncsu.edu
require known-user

Upon trying to access the HTML file, Apache should redirect you
to your webauth server, and upon authentication the webauth
server will redirect back to your HTML file. When all that is
working, you are ready to try wraplogin.module.

INSTALLATION
------------
Place the entire wraplogin module folder into your third-party modules
directory, typically at sites/all/modules.

Enable the module in Drupal by going to 
    Administer >> Site building >> Modules.

Set up wraplogin.module by going to 
    Administer >> User management >> WRAP login.

Enable the WRAP login block by going to 
    Administer >> Site building >> Blocks.

Ensure that Clean URLs are enabled at 
    Administer >> Site configuration >> Clean URLs.

LOGIN DIRECTORY
---------------
You will also need to create a WRAP-protected directory inside your 
Drupal install that Drupal will use to hook into the WRAP module 
provided by Apache. The directory should be named "wraplogin" by default
and must exist in the top level directory of your Drupal installation.
Inside the directory, create an .htaccess file as described in 
the Prerequistes section above. Do not place any other files in this
directory. (Unless you need to solve a rewrite problem, see that
section below.)

USE AUTOMATIC LOGIN
-------------------
This option is off by default. When a user visits your site, they must 
explicitly click on the WRAP login link to login to drupal. The login page
will then either use their WRAP cookie if they have one, or send them to
webauth to get one.

If you check this box, then drupal will look for the WRAP cookie on any
page where the login link would be provided. If the user already has a
WRAP cookie, they will be quietly redirected to the login page where
the cookie will be used to log the user before returning them to the
requested page.


ID/E-MAIL EQUIVALENCY
---------------------
Checking the ID/E-mail equivalency checkbox says that the distributed
login ID, such as jsmith@ncsu.edu, is also a valid email address.
In the case of WRAP, this is always true, although userid@ncsu.edu
is not necessarily the preferred email address for the user.
If this box is checked, during the registration process the wraplogin
module will insert the user's ID into the mail column of the user table.

E-MAIL DOMAIN
-------------
Use this option to change the default mail domain for ID/E-Mail equivalency.
For example, if you set this value to "gw.ncsu.edu", then all new accounts
will be setup using the email address of "unityid@gw.ncsu.edu".

DEFAULT USER ROLE
-----------------
This option was added to make it easy to identify users who have authenticated
through a standard NCSU mechanism (like WRAP) as opposed to users who have
authenticated through a local account, or external LDAP, or OpenID, or
whatever. When a user first logs in to WRAP, their account will be 
created automatically. If this option is set to a role name, the new
user account will automatically be added to that role. If the role does not
exist, it will be inserted for you. Note that this is a case sensitive
string match on the name, so it is not wise to change this option after
the site has been setup and some accounts have already been added.

LOGOUT
------
Wraplogin hooks into the user logout to destroy the user's WRAP cookie
when a logout is requested. Thus the user will logout of both Drupal 
and WRAP.

GOTCHAS
-------
If you check the ID/E-mail equivalency checkbox so the mail column of
the user table is populated, and if you have local users with the same
email address, you cannot update the accounts through
    Administer >> User management >> edit 
because Drupal says "The email address joe@example.edu is already
registered." That is because email addresses must be unique in the
user table.

Turn off caching while setting this module up for the first time.
Otherwise, the program may save a page loaded without the correct WRAP
tokens, and the users will never be able to login correctly if they keep
hitting the bad cached page. You'll find the cache settings under:
    Administer >> Site Configuration >> Performance
You can re-enable caching after the module is setup and tests out as
working properly.

WHY IT WORKS
------------
When you click on the Log In link provided by the wraplogin block, it takes
you to the directory you specified for "Login directory" under admin >
settings > wraplogin (by default, 'wraplogin'). The wraplogin module takes
this path, adds "wrap" (an arbitrary string) to the end of it and -- and
here's the key -- registers it as a menu item in the menu hook. So now
http://yourdomain.com/wraplogin/wrap is not a nonexistent file but a registered
Drupal path that is "located" inside a directory that's protected by a
.htaccess file restricting the contents to WRAP-authenticated
users. So when you reach that path, the WRAP module receives a call
to wraplogin_page() and goes from there.

DEBUGGING REWRITE PROBLEMS
--------------------------
In the OIT sites, we have the master drupal rewrite rules located in the 
server config files, not in .htaccess. In practice, that causes problems 
for this module. Namely, the way this is supposed to work is:
- user asks for URL mysite.edu/drupal/wraplogin/wrap
- apache sees that the directory "wraplogin" is WRAP protected
- user is sent to login to wrap
- user returns to mysite.edu/drupal/wraplogin/wrap
- apache sees the user has a WRAP cookie, and decodes it
- apache looks at the rewrite rule for the non-existant file,
  and does an internal redirect to /drupal/index.php?q=wraplogin/wrap
- this module hooks off that q, reads the variables, and logs the user

On out sites, the config-level rewrite is called internally to apache
BEFORE mod_auth_wrap, and so the WRAP settings on the directory are
never checked. The result is:
- user asks for URL mysite.edu/drupal/wraplogin/wrap
- apache looks at the rewrite rule for the non-existant file,
  and does an internal redirect to /drupal/index.php?q=wraplogin/wrap
- this module cannot find any wrap variables, and reports login failure

If you have this problem, this solution seems to work:
- put some dummy content in the fake wraplogin/wrap file, just to be
  sure the file does exist
- add a second rewrite rule in the wraplogin/.htaccess file so any
  request for a file in that directory is forwarded to 
  /drupal/index.php?q=wraplogin/wrap

The resulting sequence is then:
- user asks for URL mysite.edu/drupal/wraplogin/wrap
- apache looks at the server rewrite rule, finds that the target file does 
  exist, and passes thru
- apache sees that the directory "wraplogin" is WRAP protected
- user is sent to login to wrap
- user returns to mysite.edu/drupal/wraplogin/wrap
- apache looks at the server rewrite rule, finds that the target file does
  exist, and passes thru
- apache decodes the WRAP variables from the cookie
- apache look sat the .htaccess-level redirect and now redirects internally to
  /drupal/index.php?q=wraplogin/wrap
- this module hooks off that q, reads the variables, and logs the user


