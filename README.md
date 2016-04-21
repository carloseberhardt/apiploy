pushapi
=======

A bash script to import an API proxy bundle to Apigee Edge, and
optionally deploy the imported revision to a particular environment.

This allows you to edit proxy files offline, and then import and deploy
proxies remotely.

Installation
-----------

To install, copy the script to a directory. Make sure to chmod the script so that it has execute permissions. (chmod 755 pushapi)


Usage
-----

    ./pushapi [options] <API proxy directory>

The API proxy directory must contain the definition of the API proxy. For
example, if the name of the directory is `myproxy` then the structure of
that directory ought to be like this:

    myproxy/
    myproxy/apiproxy/
    myproxy/apiproxy/myproxy.xml
    myproxy/apiproxy/proxies/
    myproxy/apiproxy/proxies/...
    myproxy/apiproxy/resources/
    myproxy/apiproxy/resources/jsc/
    myproxy/apiproxy/resources/jsc/javascript1.js
    myproxy/apiproxy/resources/jsc/...
    myproxy/apiproxy/resources/xsl/
    myproxy/apiproxy/resources/xsl/file1.xsl
    myproxy/apiproxy/resources/xsl/...
    myproxy/apiproxy/policies/
    myproxy/apiproxy/policies/Policy-1.xml
    myproxy/apiproxy/policies/...
    myproxy/apiproxy/targets/
    myproxy/apiproxy/targets/...


The default action is to zip up the directory and import the resulting .zip file API proxy bundle into an Edge organization.  You can also optionally deploy the API proxy as well.

If you use a nodejs target in your proxy, the tool zips up dependent NPM modules specified in the package.json file, and sends up the entire modules.zip, within the proxy bundle.


Options
-------

     -h        display this message.
     -o org    the org to use.
     -e env    the environment to deploy to, or undeploy from.
     -d        deploy the revision. (default: no deploy). Requires environment (-e)
     -n name   override the name of the API proxy. (default: use the directory name)
     -S url    the base server url to use.
     -u creds  authn credentials for the API calls to the Edge management server.
     -c        read authn credentials from the .netrc file
     -x        undeploy any revs of the API in the environment; do not deploy a new rev.
     -X        undeploy & DELETE all revs of the API; do not deploy a new revision.
     -C num    delete all but last num revs of the API (if not deployed)
     -z        just zip the bundle; do not deploy a new revision.
     -q        quiet; decrease verbosity by 1
     -v        verbose; increase verbosity by 1



How it Works
------------

The script uses the administrative APIs exposed by Apigee Edge to perform various actions, including:

 - checking for existing revisions of an API proxy
 - checking for the deployment status of revisions of an API proxy
 - undeploying revisions
 - deleting revisions (Warning! loss of data possible!)
 - importing bundles to Edge as a new revision of an API proxy
 - deploying an imported API proxy

The script invokes these APIs using the cURL utility; it echoes the cURL commands as it runs
them, and checks the output status of those commands in order to handle
failures. You need to have curl available on the path.



Examples
--------

1. import an API proxy called xyz-xform to the org named 'demo11':

    ./pushapi -c -o demo11 xyz-xform

  The API proxy will get a new revision. The output of the script will
  tell you the revision number. This will not deploy the proxy.

  In this case there should be a directory called xyz-xform in the
  current directory, and it should contain exactly one
  subdirectory, called `apiproxy`, and in there should be the requisite
  Apigee Edge proxy directory tree, containing all the files in your proxy.

2. import an API proxy called xyz-xform to the org named 'demo11', and deploy it to the environment named 'test':

    ./pushapi -c -o demo11 -d -e test  xyz-xform

  This will create a new revision of the API proxy.  The output of the script will tell you the revision number.
  To deploy a proxy, you need to specify both the -d option (which says "please deploy"), and the -e option (which
  says which environment to deploy to). 


3. import an API proxy called xyz-xform to the org named 'demo11', and deploy it to the environment named 'test', with verbose output:

    ./pushapi -c -v -o demo11 -d -e test xyz-xform

  There is also a -q option for "quiet" operation.  By default, if you
  use two -q's then you get "silent" operation. If you use additional
  -v's then you get more verbosity.

2. import an API proxy, sourcing from the directory called ~/foo/bar/xyz-xform. The proxy will be named 'newname' and will be imported to the org named 'demo11'. It will be deployed to the environment named 'test':

    ./pushapi -c -o demo11 -d -e test -n newname  ~/foo/bar/xyz-xform

  This will create a new revision of the API proxy.  The output of the script will tell you the revision number.


4. just create the API proxy zip bundle from the given directory; don't import or deploy:

    ./pushapi -z ~/foo/bar/xyz-xform

5. undeploy any deployed revisions of the API proxy called abc from any environment in the demo11 org:

    ./pushapi -c -o demo11 -x abc

7. undeploy any deployed revisions of the API proxy called abc from the environment test in the demo11 org:

    ./pushapi -c -o demo11 -e test -x abc

6. undeploy any deployed revisions of the API proxy called abc from any environment in the demo11 org, and then DELETE ALL REVISIONS:

    ./pushapi -c -o demo11 -X abc

8. delete all but the most recent 10 revisions of the mayo1 API proxy:

    ./pushapi -c -o demo11 -C 10 mayo1



Default Settings
----------------

The file .pushapi can contain settings for the org, env,
credentials, verbosity and other options. This file is dot-sourced from
within pushapi itself. This can eliminate the need for you to
specify all the options each time you invoke the script. Don't
put syntax errors in this settings file.

Example contents:

    org=demo11
    environment=test
    credentials=marjorie:secr3t
    verbosity=1

You can set quiet operation in the default settings file with:

    verbosity=1

Or specify silent operation with:

    verbosity=0

If you use git, you should specify the file .pushapi in your .gitignore file to
avoid uploading secrets to the git repo.

Also, you can eliminate the credentials from the .pushapi file by using
the -c option and placing the credentials in the .netrc file.


Using this with OPDK
------------------------

This works against Apigee Edge public cloud or Apigee Edge OPDK.  The way you specify the management server target is with the -S option.  It defaults to https://api.enterprise.apigee.com today, but you can point it to a local OPDK via something like

    ./pushapi -c -S http://api.edgemgmt -o demo11 -C 10 mayo1

...and in this case the urls the script will use will be like

    http://api.edgemgmt/v1/o/demo11/...


Credentials
------------------------

You need to pass your administrative credentials to the script in some way. Use one of these three ways:

  - the -u option, specifying the creds on the command line
  - the -c option, telling the script to look in .netrc.  Curl actually can
    look in the .netrc file for your creds, so this is a nice option if
    you don't want to constantly have to specify credentials.
  - in the .pushapi file, as a default setting.

Most of the examples above show the use of the -c option.  You could replace -c in any of those examples with -u username:password .



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
