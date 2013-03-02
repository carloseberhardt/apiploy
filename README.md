pushapi
=======

A bash script to deploy an API proxy definition to the Apigee
Gateway.


Installation
-----------

To install, copy the script to the directory that holds directories that contain api proxy definition projects.


Usage
-----

    ./pushapi [options] <apiproxy directory>


The apiproxy directory must contain the definition of the API proxy. For
example, if the name of the directory is "myproxy" then the structure of
that directory ought to be like this:

    myproxy/
    myproxy/apiproxy/
    myproxy/apiproxy/myproxy.xml
    myproxy/apiproxy/proxies/
    myproxy/apiproxy/proxies/...
    myproxy/apiproxy/resources/
    myproxy/apiproxy/resources/...
    myproxy/apiproxy/stepdefinitions/
    myproxy/apiproxy/stepdefinitions/...
    myproxy/apiproxy/targets/
    myproxy/apiproxy/targets/...

The pushapi script uses the name of the toplevel directory as the
name of the API proxy bundle.

The options allow you to specify an environment, an organization,
credentials, and a revision number for deployment. The relevant command line
options are -e -o -c and -r respectively.

Example: To deploy an API called xyz-xform to the test environment in the demo11 org:

    ./pushapi  -o demo11 -e test xyz-xform

In this case there should be a directory called xyz-xform in the
current directory, and it should contain exactly one
subdirectory, called apiproxy, and in there should be all the
relevant apigee Gateway proxy definition files.

There is also a -q option for "quiet" operation.  By default, you
use two -q's then you get "silent" operation. There is a
complementary -v option to increase verbosity. The -v option
works only if you have set verbosity lower than 2, with the
default settings file. More about that file later.


How it Works
------------

The sript does this:

 - Check for an existing revision with the number given
 - undeploys and deletes that revision if it exists. (Warning! loss of data possible!)
 - creates the zip for the new API bundle
 - imports the bundle to the gateway
 - deploys the bundle
 - deletes the just-uploaded zip

It uses cURL for the commands; it echoes the cURL commands as it runs
them, and checks the output status of those commands in order to handle
failures.


Default Settings
----------------

The file .pushapi can contain settings for the org, env,
credentials, revision, and verbosity. This file is sourced from
within pushapi itself. This can eliminate the need for you to
specify all the options each time you invoke the script. Don't
put syntax errors in this settings file.

Example contents:

    org=demo11
    environment=test
    credentials=scott:tiger
    rev=1
    verbosity=1

You can set quiet operation in the default settings file with:

    verbosity=1

Or specify silent operation with:

    verbosity=0

You should specify the file .pushapi in your .gitignore file to
avoid uploading secrets to the git repo.


Known Bugs
----------

None, as far as I know.


Testing
-------

What, me worry?


Contributing
------------

1. Fork it.
2. Create a branch (`git checkout -b my_improvement`)
3. Commit your changes (`git commit -am "Added Awesumo!"`)
4. Push to the branch (`git push origin my_improvement`)
5. Open a pull request.
6. Share and enjoy.
