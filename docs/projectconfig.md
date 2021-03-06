# Note
This file is a work in progress, and contains open questions. At the same time it is used as a reference while building,
together with the projectconfig.sample.json file.
If you find inconsistencies or just want to send remarks, feel free to leave comments on this file, file an issue, or send a pull request!

# Configuration Files

## Main concept

The buildtools is an application written in Sproutcore. This also means that the configuration files are code files which are evaluated
on startup.


### Environment

Because it is an SC application load order is important. The first file that will be evaluated is the project config file.

The environment provides you with the following globals:

- SC: a reference to SproutCore itself
- BT: a reference to the BuildTools application
- sc_require: a function which allows you to load and execute external sc_config files in the project

#### sc_require

The sc_require function will inject and execute the code inside the referred file in place. This means that if you want to refer to a global
variable in the different file, you have to declare it before calling sc_require:


apps/myapp/sc_config
```
MyProject.myApp = BT.AppBuilder.create({ path: BT.curPath });

```

sc_config
```
MyProject = SC.Application.create();

sc_require('apps/myapp');
```

#### BT

The BT application exposes the following API:

- BT.addApp: registers an application to the BuildTools. Usually you won't have to call this, as BT.AppBuilder.create() will automatically register itself with the BuildTools. BT.addApp will however allow a single string.
- BT.addFramework: registers a framework to the BuildTools with a specific configuration for this project. In this way you can define certain properties that will override default settings of a framework for the entire project. Be aware that you need to register frameworks before they
are declared a dependency in your application

The BT application exposes a few extra properties and functions

- BT.curDir: The directory of the sc_config file being evaluated
- BT.curFile: The full path of the sc_config file being evaluated
- BT.projectDir: The path to the root of the project




### Project config

In this file you need to either define your applications or use sc_require to load them from a different file.
As written above, this file is the first file to be evaluated, so if you


### Application config


### Framework config




















# Old information
The information below is kept in the file for now, because it does contain important definitions of internal objects and things to configure.

## Main concept

The buildtools use SC classes to generate functionality on the level of the project, app and framework.
In order to make the configuration match nicely with both SC as well as make the configuration as consistent as possible throughout, the configuration should be very similar to the way default properties of SC classes are overriden, namely through hashes in the create statement.

The buildtools can then do something like this:

```var fw = Framework.create(fwconfig,globalconfig,appconfig)```, where
  - fwconfig is the internal configuration file of the framework itself
  - globalconfig is the framework configuration found in the frameworks section of the project configuration file
  - appconfig is the configuration found in the frameworks section of the app where this framework is being added to
  - when deploying, the deploy configuration is added as fourth parameter, if present.

which will allow a default configuration in the framework itself, additional configuration on project level, as well as app-specific overrides.
As will be shown later, every framework can and will have multiple object instances, in order to allow app specific overrides as well as to prevent app specific overrides leaking into other apps.

The example project config file contained in this directory is very complex in order to show all the possible overrides. Configuration files found inside apps or frameworks should have the same syntax as the corresponding parts in the project config.
In a real project, the project configuration file can most likely be very small, or even non-existent as the buildtools will try to autodetect the project configuration as much as possible.

All config files for the SproutCore buildtools are called sc_config.json

## Project configuration file sections

There are different sections in the project config.

### Server
The server section in the configuration contains all the settings intended to be used by the development server.
It is a hash containing essentially three items: proxies, port and allowFromAll

#### Proxies
When developing a SproutCore app, you might want to use images or other data that should not be included in the app itself. To make the development easier, the buildtools development server can proxy requests to a different server. A proxy configuration contains:

  1. prefix: the first part of the url the server should match against. "/images" will trigger this proxy for all urls starting with "/images"
  2. host: to which host should the proxy forward the request
  3. port: on which port should the proxy forward the request
  4. proxyPrefix: How should the proxy change the url of the forwarded request. If empty it will forward the request as is, if not empty it will replace the prefix given at 1. with the string in this setting.

#### Other settings

  1. port: on which port should the devserver run. Default is 4020
  2. allowFromAll: if true: allow requests from other computers than localhost.

### Plugins

The plugins section is to configure included plugins or additional plugins inside the project for all apps and all frameworks.
It is a hash of which the key is the name of the plugin to be used, and the value is a hash containing

  1. type: which is the type of file targeted by this plugin (scripts, stylesheets or resources)
  2. extension: which file extension this plugin should be acting on?
  3. node_module: (optional) which node module should be required?

If you do not define a node_module value, the buildtools assume that the name of the plugin is the module to require();

### Apps and Frameworks
The apps and frameworks sections can have two syntaxes:

  1. the sections are arrays, containing strings with references (sproutcore:desktop). If a certain framework should not be loaded, the reference should start with a !
  2. the sections are hashes in which the key is the name of the app or framework (if a subframework is intended, a syntax like "sproutcore:desktop" as name), and the value a hash with properties. If this hash does not contain a key named path, and the autodetection did not pick up any configuration declaring this name (which can also be like "sproutcore:desktop"), it is assumed the key is equal to a folder name in apps/ or frameworks/. Property values in these hashes will override configurations found inside these apps or frameworks. More about the specifics for app and framework configurations in their respective chapters. For frameworks there are two special options "before" and "after", which both are arrays and which indicate whether the framework should be loaded before or after a certain other framework.


### Deploy

The deploy section can contain settings which overrides all other configurations, but only in case of a build (or save).
The section can have a frameworks and an apps section, describing for which app or framework what setting should be overridden.
When a deploy action is run, the configuration will be added after all the others, overriding any other setting made at any other place for this particular framework or app.
TODO: describe here the default deploy settings (such as minifyScripts?)

## Framework configuration file fields

  - name: The name section contains the name of the framework (optional). The name can either be set in the configuration, or it is derived from the path. If set, complex notation can be used: "sproutcore:desktop".
  - path: The path section contains the relative path of the framework. This can be either set, or computed from the name. If the path is a filename, the file will be included. If the path is an url, the url will be inserted in the generated html file.
  - combineScripts: set to true if scripts of this framework should be combined into one file (default: true)
  - combineStylesheets: set to true if stylesheets of this framework should be combined into one file (default: true)
  - minifyScripts: set to true if scripts of this framework should be minified (default: false)
  - minifyStylesheets: set to true if stylesheets of this framework should be minified (default: false)
  - stylesheetProcessor: which processor should be used for the stylesheets in this framework
  - watchForChanges: set to true if the current framework should be watched for changes
  - dontProcess: set to true if the buildtools should include the code found here as is
  - dependencies: an array of strings with references pointing at the runtime dependencies
  - testDependencies: an array of strings with references pointing at dependencies needed to run the tests

The settings dontProcess together with the path being a file can be used to inject certain code on a certain spot in the load order.
An example for this use could be that a custom localization should be loaded externally, but before the app itself loads in order to
not have to rebuild the entire application whenever a translation string changes.

The following might need a separate class, extending from Framework, perhaps because of specific requirements:

  - isModule: set to true if the current framework should be treated like a module (default: false)
  - shouldPreload: set to true if the current module (isModule needs to be true) should be packed together with the main package (default: false)

## Application configuration file fields

  - name: contains the name of the application. If left empty, the buildtools will get the folder name from the path
  - path: contains the path of the application
  - theme: which theme the app uses
  - buildLanguage: tells the build tools which language should be built (deprecate?)
  - htmlHead: can be either a string or array containing additional tags to be included in the main html
  - htmlBody: if set this will be used as loading page, to which things will be added SC requires in order to start.
  - urlPrefix: if set, all urls generated for this application will be prefixed with this url. For example:
    "" will cause a relative build, "/" will make the urls absolute.
  - includeSC: if set, automatically includes the SproutCore framework. If frameworks/sproutcore doesn't exist,
    it will use the internal version (default: true)
  - useSCFrom: override to decide for the app which sproutcore should be used, should be a path, only use when the default locations cannot be used, or you want to use a different sproutcore version for this specific app
  - include: if set, the app will be included for serving and / or deploying (building) (default: true)
  - dependencies: this is either an array with framework references, or a hash conform the apps/frameworks hash described above. A framework reference can also be a path, but also an url. In case sproutcore is detected in the list, the app will respect the order of the dependencies and inject frameworks or files before it adds sproutcore.

As an app itself is very much like a framework with extras, an app will automatically add itself as last framework.
The following configuration fields can be used to configure that framework.

  - combineScripts: set to true if scripts of this framework should be combined into one file (default: true)
  - combineStylesheets: set to true if stylesheets of this framework should be combined into one file (default: true)
  - minifyScripts: set to true if scripts of this framework should be minified (default: false)
  - minifyStylesheets: set to true if stylesheets of this framework should be minified (default: false)
  - stylesheetProcessor: which processor should be used for the stylesheets in this framework


## References
There is a codified option of referring to frameworks and subframeworks, as well as to other sources.
These references can have the following formats
  1. "sproutcore": depending on the context this is either an app, a framework or a module in the root of the project
  2. "sproutcore:desktop": this is the subframework desktop inside the sproutcore framework
  3. "sproutcore/lib/index.html": this is a reference to the file lib/index.html inside the sproutcore framework
  4. "http://my.host.ext": a url, is taken literally


# Different API
As mentioned earlier (perhaps not in this document) typing JSON is not very easy nor convenient. Looking at a way to convert the old Buildfile format I noticed that the Abbot Buildfiles are actually plain Ruby, and are executed as such. The Buildfile contains a function call to config which inserts the configuration into the internal configuration tree.
This approach is also very feasible for these buildtools. Because the parse tree for the configuration will be very similar anyhow, it can be an addon, on which the community can decide which format to support in the end.
The files will be called sc_config (without .js), but will be loaded and executed as JS.
