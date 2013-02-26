pushapi
=======

A bash script to deploy an API proxy definition to the Apigee
Gateway.


Installation
-----------

To install, copy the script to the directory that holds directories that contain api proxy definition projects..


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


How it Works
------------

The sript does this:

 - Check for an existing revision with the number given
 - undeploys and deletes that revision if it exists. (Warning! loss of data possible!)
 - creates the zip for the new API bundle
 - imports the bundle to the gateway
 - deploys the bundle
 - deletes the just-uploaded zip

It uses cURL for the commands; it echoes the curl commands as it runs them, and checks the status upon failure.


Default Settings
----------------

The file pushapiDefaults can contain settings for the org, env,
credentials and revision. This file is sourced from within
pushapi itself. This can eliminate the need for you to specify
all the options each time you invoke the script.

Example contents:

    org=demo11
    environment=test
    credentials=scott:tiger
    rev=1


You should specify the file pushapiDefaults in your .gitignore file to
avoid uploading secrets to the git repo.



Known Bugs
----------

Does not echo error messaged to stderr.


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
