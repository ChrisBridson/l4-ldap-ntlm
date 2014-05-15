Changes
============
Fork of https://github.com/wells/l4-ldap-ntlm 
Additions:
* Authentication bug fixes
* Modifications to config file processing
** Dynamic groups with differing levels
** Changed 'type' key to 'level' ro represent access level


l4-ldap-ntlm
============

An LDAP/Active Directory/NTLM authentication driver for Laravel 4.

This package will enable you to have basic authentication with a config-based ACL for admin and viewers of any auth based portion of a Laravel 4 based site. In addition, the package is capable of tying into Apache based NTLM authentication. You will need to install and configure both `php5-ldap` and `libapache2-mod-auth-ntlm-winbind` for Apache2 (Visit http://goo.gl/cvCMla for a tutorial). If it is not installed, the package should still operate.

Installation
============

To install this in your application add the following to your `composer.json` file

```json
require {
	"ChrisB/l4-ldap-ntlm": "dev-master"
}
```

Then run `composer install` or `composer update` as appropriate

Once you have finished downloading the package from Packagist.org you need to tell your Application to use the LDAP service provider.

Open `app/config/app.php` and add:

`ChrisB\L4LdapNtlm\L4LdapNtlmServiceProvider`

This tells Laravel 4 to use the service provider from the vendor folder.

You also need to direct Auth to use the ldap driver instead of Eloquent or Database. 

Edit `app/config/auth.php` and change driver to `ldap`

Configuration
=============

Add the following config into your `app/config/auth.php` file

```js
/**
 * LDAP Configuration for ChrisB/l4-ldap-ntlm
 */
'ldap' => array(
	// Domain controller (host), Domain to search (domain), 
	// OU containing users (basedn), OU containing groups (groupdn)
	'host' => 'dc',
	'domain' => 'domain.com',
	'basedn' => 'OU=Users,DC=domain,DC=com',
	'groupdn' => 'OU=Groups,DC=domain,DC=com',

	// Domain credentials the app should use to access DC
	// This user doesn't need any privileges
	'dn_user' => '*',
	'dn_pass' => '*',

	//At minimum, you'll need these attributes
	'attributes' => array(
		'dn', 
		'samaccountname',
		'memberof'
	),

    // New dynamic group structure, can have any number of groups
    // level denotes a number/string for use in your app as neccessary.
    'group_levels' => array(
            'users' => array(
                'level'=>0,
                'description'=>'General Users',
                'groups'=>array(
                        'group1',
                        'CN=group2,OU=AnotherOU,OU=Groups,DC=domain,DC=com',
                    ),
            ),
            'super_users' => array(
                'level'=>1,
                'description'=>'Users with additional privs.',
                'groups'=>array(
                    'group3',
                    'CN=group4,OU=Super,OU=Groups,DC=domain,DC=com',
                ),
            ),
            'admins' => array(
                'level'=>2,
                'description'=>'The Admin Group',
                'groups'=>array(
                        'group5',
                        'CN=group6,OU=Admins,OU=Groups,DC=domain,DC=com',
                ),
            ),
            // Denote username by the 'user' key
            'owners' => array(
                'level'=>10,
                'user'=>true,
                'description'=>'The Owners',
                'groups'=>array('userxyz','userabc')
            ),
    ),


),
```

Usage
======

In addition to the default Auth functionality, You can enable NTLM authentication with the auto() method from provided Guard class. Edit `app/config/filters.php` and change to:

```js
Route::filter('auth', function()
{
	// !Auth::user() checks to see if the user has access permission
	if (!Auth::auto() || Auth::guest()) return Redirect::guest('login');
});
```

