# Purpose
Openshift already has [Node.js](http://nodejs.org/) cartridge.  So why use this one?

* [Grunt](http://gruntjs.com/) support
* [Bower](http://bower.io/) support
* Adds --production so devDependencies aren't pulled in
* Up to date with the latest .10.x release (Currently using 0.10.26)
* Not every OpenShift Enterprise is going to have Node.js as a default cartridge.

# Usage

To deploy this cartridge with the cartridge reflector you can execute the following command
`rhc create-app <app name> "http://cartreflect-claytondev.rhcloud.com/reflect?github=bogdanbrudiu/openshift-origin-cartridge-nodejs"`

## Template Repository Layout

    node_modules/            Any Node modules packaged with the app [1]
    deplist.txt              Deprecated.
    package.json             npm package descriptor.
    Gruntfile.js			  Optional Grunt configuration, `grunt prod` is executed in ./bin/control
    bower.json				  Optional bower configuration, `bower install` is executed in ./bin/control
    npm_global_module_list   List of globally installed node modules (on OpenShift)
    .openshift/              Location for OpenShift specific files
      action_hooks/          See the Action Hooks documentation [2]
      markers/               See the Markers section [3]

\[1\] [`node_modules`](#node_modules-directory)
\[2\] [Action Hooks documentation](https://github.com/openshift/origin-server/blob/master/node/README.writing_applications.md#action-hooks)
\[3\] [Markers](#markers)

### Layout Notes

Please leave the `node_modules` and `.openshift` directories but feel free to
create additional directories if needed.

Note: Every time you push, everything in your remote repo dir gets recreated
      please store long term items (like an sqlite database) in the OpenShift
      data directory, which will persist between pushes of your repo.
      The OpenShift data directory is accessible relative to the remote repo
      directory (`../data`) or via an environment variable `$OPENSHIFT_DATA_DIR`.

#### `node_modules` directory
The `node_modules` directory allows you to package any Node module on which
your application depends along with your application.

If you just wish to install module(s) from the npm registry
([npmjs.org](https://npmjs.org/)), you
can specify the module name(s) and versions in your application's
`package.json` file.


#### package.json

npm package descriptor - run `npm help json` for more details.

Note: Among other things, this file contains a list of dependencies
      (node modules) to install alongside your application and is processed
      every time you `git push` to your OpenShift application.


## Environment Variables

The Tomcat cartridge provides several environment variables to reference for ease
of use:

    OPENSHIFT_NODEJS_IP     The IP address used to bind Node.js
    OPENSHIFT_NODEJS_PORT   The Node.js listening port

For more information about environment variables, consult the
[OpenShift Application Author Guide](https://github.com/openshift/origin-server/blob/master/node/README.writing_applications.md).


## Markers

Adding marker files to `.openshift/markers` will have the following effects:

    hot_deploy          Disable app restarting during git pushes (see 'Development Mode')


## Development Mode

When you push your code changes to OpenShift, if you want dynamic reloading
of your javascript files in "development" mode, you can either use the
`hot_deploy` marker or add the following to `package.json`:
   
    "scripts": { "start": "supervisor <relative-path-from-repo-to>/server.js" },

This will run Node with Supervisor - https://npmjs.org/package/supervisor


## Local Development + Testing

You can also develop and test your Node application locally on your machine
(workstation). In order to do this, you will need to perform some
basic setup - install Node + the npm modules that OpenShift has globally
installed:
   1. Collect some information about the environment on OpenShift.
         A. Get Node.js version information:
        $ ssh $uuid@$appdns node -v
         B. Get list of globally install npm modules
        $ ssh $uuid@$appdns npm list -g

   2. Ensure that an appropriate version of Node is installed locally.
      This depends on your application. Using the same version would be
      preferable in most cases but your mileage may vary with newer versions.

   3. Install the versions of the Node modules you got in step 1.A
      Use -g if you want to install them globally, the better alternative
      though is to install them in the home directory of the currently
      logged user on your local machine/workstation.
         pushd ~
         npm install [-g] $module_name@$version
         popd


Once you have completed the above setup, you can then run your application
locally by using any one of these commands:
    node server.js
    npm start -d
    supervisor server.js

And then iterate on developing+testing your application.
