1. Introduction
===============

_WebIDDelegatedAuth_ is a lite version of _libAuthentication_ (<https://github.com/melvincarvalho/libAuthentication>), 
which is a PHP implementation of the WebID protocol. _WebIDDelegatedAuth_ is only 
used for delegated WebID authentication. All credit belongs to the initial authors 
of _libAuthentication_.

Further details of WebID can be obtained at <http://webid.info>

If you would like to learn how to get going quickly without diving to much into
technical details, then read section 2. and 3.

The core classes of _WebIDDelegatedAuth_ are tackled in section 3. and 4.

--------------------------------------------------------------------------------

2. How to set up WebID authentication in a few lines of code
================================================================================

There are a few flavours of Foaf+SSL authentication. The following very simple
example shows how to setup a Foaf+SSL authentication relying on an identity 
provider such as foaf-ssl.org.

Prerequisites:

  *   Publicly available internet site
  *   Apache 2.2 and PHP 5.2.x or higher

Checkout and create a script that will be the entry point for your application:

    git clone git://github.com/melvincarvalho/libAuthentication.git

    cat > index.php
    <?php

    require_once('WebIDDelegatedAuth/lib/Authentication.php');
    $auth = new Authentication_Delegate();

    if (!$auth->isAuthenticated()) 
    { 
      echo $auth->authnDiagnostic;
      echo '<a href="https://foafssl.org/srv/idp?authreqissuer=http://localhost/index.php">Click here to Login</a>';
    } 
    else 
    { 
      echo 'Your have succesfully logged in.<pre>';
      print_r($auth);
    } 

Make sure the _"authreqissuer"_ points to YOUR site and...
... YOU ARE DONE!

You just set up you first Foaf+SSL powered site. Behind the scenes,
_WebIDDelegatedAuth_ has a copy of foafssl.org's certificate which is used
in the authentication process.


Note that if you wish to use another delegated identity verification
service (for instance 'auth.my-profile.eu'), you maye need to change line 4 as :

    $auth = new Authentication_Delegate(TRUE, NULL, Authentication_URL::parse('https://auth.my-profile.eu'));

Then you'd change the login link to : 

    echo '<a href="https://auth.my-profile.eu/auth/?authreqissuer=http://localhost/index.php">Click here to Login</a>';

This will ensure that you wish to verify the server's response
signature according to the proper certificate already present in Authentication_X509CertRepo.php

--------------------------------------------------------------------------------

3. Brief overview of _WebIDDelegatedAuth_'s core classes
================================================================================

_WebIDDelegatedAuth_ provides the following core classes:

*   Authentication
    Authenticate user by trying the supported authentication methods in a fixed 
    and reasonable sequence

*   Authentication_Delegate
    Authenticate via the delegated WebID method using a 3rd party WebID 
    identity provider (foafssl.org and auth.my-profile.eu supported by default)

*   Authentication_Session
    Create a session cookie after successful authentication to speed up 
    subsequent authentication attempts

A detailed description of the core classes an their usage follows.

--------------------------------------------------------------------------------

4. Detailed description of _WebIDDelegatedAuth_'s core classes
================================================================================

class Authentication
--------------------------------------------------------------------------------
This class provides easy access to all supported authentication mechanisms. 
On instantiation, it performs the following operations:

1.  Checks if an authentication session cookie is present
2.  If 1. fails, it tries to authenticate via delegated WebID (see _Authentication\_Delegate_)
3.  If authentication is successful, it loads the corresponding foaf file

        $auth = new Authentication($config) // $config is optional,  
                                            // only necessary if ARC is used

On Success:

-   `$auth->isAuthenticated()` returns true
-   `$auth->webid` contains the authenticated webid

On Error:

If an error occurs, an explanation can be retrieved by inspecting
`$auth->$authnDiagnostic`.
If you want to terminate the authenticated session, it is a good idea to call
`$auth->logout`.

class Authentication_Session
--------------------------------------------------------------------------------

This class usually won't be instantiated directly. If a given authentication 
method succeeds, it can optionally persist that information by instantiating 
_Authentication\_Session_. It stores the authenticated webid and the parsed foaf
file in `$_SESSION`. This results in a significant speed up in successive 
authentication attempts. If you want to create it manually, you can do that as follows:

    $authSession = new Authentication_Session( 1, $webid)

where 1 indicates the fact of successful authentication and `$webid` is a URI string.

class Authentication_Delegate
--------------------------------------------------------------------------------

Using the delegated WebID method is probably the easiest way to get you start 
quickly leveraging this powerful authentication method. It is also the easiest 
to set up. Refer to Section 2. for an example and make sure you set up the example 
using a public domain name or a public IP address. I you want find out more details 
how the identity provider works, see <https://foafssl.org/srv/idp>.

You need to instantiate _Authentication\_Delegate_ at a common entry point 
to your site (e.g. index.php):

    $auth = new Authentication_Delegate();

Most of the input is automatically retrieved from the global php context variables 
(`$_REQUEST`, `$_SERVER` etc.), so using the default constructor parameters is fine. 

On Success:

-   `$auth->isAuthenticated()` returns true
-   `$auth->webid` contains the authenticated webid

If not explicitly disabled, on successful authentication an instance of 
_Authentication\_Session_ will also be created, to speed up further authentication 
attempts. If that something you don't want to happend, you need to call the constructor 
as follows:

    $auth = new Authentication_Delegate( false );

On Error:

If an error occurs, an explanation can be retrieved by inspecting `$auth->$authnDiagnostic`.
If you want to terminate the authenticated session, it is a good idea to call `$auth->logout`.

