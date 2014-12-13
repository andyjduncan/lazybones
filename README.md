Lazybones Project Creation Tool
===============================

Lazybones was born out of frustration that [Ratpack](http://ratpack-framework.org/)
does not and will not have a command line tool that will bootstrap a project.
It's a good decision for Ratpack, but I'm lazy and want tools to do the boring
stuff for me.

The tool is very simple: it allows you to create a new project structure for
any framework or library for which the tool has a template. You can even
contribute templates by sending pull requests to this GitHub project or publishing
the packages to the relevant [Bintray repository](https://bintray.com/repo/browse/pledbrook/lazybones-templates)
(more info available below).

[![Build Status](https://drone.io/github.com/pledbrook/lazybones/status.png)](https://drone.io/github.com/pledbrook/lazybones/latest)

## Developers

* [Peter Ledbrook](https://github.com/pledbrook)
* [Kyle Boon](https://github.com/kyleboon)
* [Tommy Barker](https://github.com/tbarker9)

## Contributors

* [Luke Daley](https://github.com/alkemist)
* [Tomas Lin](https://github.com/tomaslin)
* [Russell Hart](https://github.com/rhart)
* [Dave Syer](https://github.com/dsyer)
* [Andy Duncan](https://github.com/andyjduncan)


## Running it

Grab lazybones from [gvm](http://gvmtool.net):

    gvm install lazybones

or alternatively, grab the distribution [from Bintray](https://bintray.com/pkg/show/general/pledbrook/lazybones-templates/lazybones),
unpack it to a local directory, and then add its 'bin' directory to your `PATH`
environment variable.

### Creating projects

To create a new project, run

    lazybones create <template name> <template version> <target directory>

So if you wanted to create a skeleton Ratpack project in a new 'my-rat-app'
directory you would run

    lazybones create ratpack-lite 0.1 my-rat-app
    
The version is optional and if you leave it out, Lazybones will install the
latest version of the template it can find. 

Named templates are all stored on Bintray. By default, Lazybones searches for
templates in the pledbrook/lazybones-templates repository, but you can use
other Bintray repositories by adding some configuration - set the Custom
Repositories section under Configuration later in this document.

You're not limited to only Bintray as you can install templates directly from 
a URL too:

    lazybones create http://dl.bintray.com/kyleboon/lazybones/java-basic-template-0.1.zip my-app

Of course it can be pretty laborious copying and pasting URLs around, so Lazybones
allows you to configure aliases for URLs that you use frequently. By adding the
following configuration to your Lazybones settings file, `~/.lazybones/config.groovy`
(see below for more details on this), you can install the template by name:

    templates {
        mappings {
            myTmpl = "http://dl.bintray.com/..."
        }
    }

In other words, you could now run

    lazybones create myTmpl my-app

Note that when using the URL option, there is no need to specify a version. You
should also be aware that mappings take precedence, i.e. if a mapping has the
same name as an existing template, the mapping is used. This essentially creates
a simple override mechanism.

There is just one more thing to say about the `create` command: by default it
creates the specified directory and puts the initial project in there. If you
want to unpack a template in the current directory instead, for example if you
have already created the project directory, then just pass '.' as the directory:

    lazybones create ratpack .

Once you have created a new project from a template, you may notice that the
project directory contains a .lazybones sub directory. You may delete this, but
then you won't be able to use the `generate` command (see next section) if the
project template has support for it.

Many project templates request information from you, such as a project name, a
group ID, a default package, etc. If this is the umpteenth time you have created
a project from a given template, then answering the questions can become tedious.
There is also the problem of scripting and automation when you want to create
a project without user intervention. The solution to both these issues is to
pass the values on the command line:

    lazybones create ratpack 0.2 ratapp -Pgroup=org.example -Ppackage=org.example.myapp

The `-P` option allows you to pass property values into the project templates
without user intervention. The key is to know what the property names are, and
that comes down to the project template. At the moment, the best way to find out
what those properties are is to look at the post-install script itself.

The last option to mention is `--with-git` which will automatically create a
new git repository in the project directory. The only requirement is that you
have the `git` command on your path.

### Sub-templates

As of Lazybones version 0.7, project templates can incorporate sub-templates.
Imagine that you have just created a new web application project from a template
and that template documents that you can create new controllers using a sub-
template named `controller`. To use it, just `cd` into the project directory
and run

    lazybones generate controller
    
This will probably ask you for the name of the controller and its package before
generating the corresponding controller file in your project. You can reuse the
command to create as many controllers as you need.

As with the `create` command, you can also pass in property values on the command
line if the sub-template is parameterised:

    lazybones generate controller -Ppackage=org.example.myapp -Pclass=Book
    
The last option available to you as a user is template qualifiers. These only
work if the sub-template supports them, but they allow you to pass additional
information in a concise way:

    lazybones generate artifact::controller
    
In this case, the template name is `artifact`, but we have qualified it with
an extra `controller`. You can pass in as many qualifiers as you want, you just
separate them with `::`.

Note that you do not specify a version with the `generate` command. This is
because the sub-templates are embedded directly in the project template, and
so there can only be one version available to you.

### Finding out what templates are available

To see what templates you can install, run

    lazybones list
 
This will list all aliases and remote templates. If you want to see what
templates you have cached locally, run

    lazybones list --cached

In fact, `--cached` is implied if Lazybones can't connect to the internet.

You can also find out more about a template through the `info` command:

    lazybones info <template name>

This will print a description of the template and what versions are available
for it. If you're offline, this will simply display an error message.

## Configuration

Lazybones will run out of the box without any extra configuration, but you can
control certain aspects of the tool through the configuration file
`~/.lazybones/config.groovy`. This is parsed using Groovy's `ConfigSlurper`, so
if you're familiar with that syntax you'll be right at home. Otherwise, just see
the examples below.

### Custom repositories

Lazybones will by default download the templates from a specific BinTray
repository. If you want to host template packages in a different repository
you can add it to Lazybone's search path via the `bintrayRepositories`
setting:

    bintrayRepositories = [
          "kyleboon/lazybones",
          "pledbrook/lazybones-templates"
    ]

If a template exists in more than one repository, it will be downloaded from the
first repository in the list that it appears in.

### General options

These are miscellaneous options that can be overridden on the command line:

    // <-- This starts a line comment
    // Set logging level - overridden by command line args
    options.logLevel = "SEVERE"

The logging level can either be overridden using the same `logLevel` setting:

    lazybones --logLevel SEVERE info ratpack

or via `--verbose`, `--quiet`, and `--info` options:

    lazybones --verbose info ratpack

The logging level can be one of:

* OFF
* SEVERE
* WARNING
* INFO
* FINE
* FINEST
* ALL

## Building it

This project is split into two parts:

1. The lazybones command line tool; and
2. The project templates.

### The command line tool

The command line tool is created via Gradle's application plugin. The main
class is `uk.co.cacoethes.lazybones.LazyBonesMain`, which currently implements
all the sub-commands (create, list, etc.) as concrete methods.

The main class plus everything else under src/main is packaged into a lazybones
JAR that is included in the distribution zip. The application Gradle plugin
generates a `lazybones` script that then runs the main class with all required
dependencies on the classpath.

To build the distribution, simply run

    ./gradlew distZip

### The project templates

The project templates are simply directory structures with whatever files in
them that you want. Ultimately, the template project directories will be zipped
up and stored on [Bintray](https://bintray.com/repo/browse/pledbrook/lazybones-templates).
From there, lazybones downloads the zips on demand and caches them in a local
user directory (currently ~/.lazybones/templates).

If you want empty directories to form part of the project template, then simply
add an empty .retain file to each one. When the template archive is created,
any .retain files are filtered out (but the containing directories are included).

To package up a template, simply run

    ./gradlew packageTemplate<TemplateName>

The name of the project template comes from the containing directory, which is
assumed to be lowercase hyphenated. The template name is the equivalent camel
case form. So the template directory structure in src/templates/ratpack-lite
results in a template called 'RatpackLite', which can be packaged with

    ./gradlew packageTemplateRatpackLite

The project template archive will be created in the build directory with the
name '<template name>-template-<version>.zip'. See the small section below on
how the template version is derived.

You can also package all the templates in one fell swoop:

    ./gradlew packageAllTemplates

Once a template is packaged up, you can publish it to a generic (non-Maven)
Bintray repository by running

    ./gradlew publishTemplate<TemplateName>

This will initially fail, because the build does not know where to publish to.
That's quickly fixed by adding a gradle.properties file in the root of this
project that contains at least these properties:

    repo.username=your_bintray_username
    repo.apiKey=your_bintray_apikey

You can then publish new versions of templates whenever you want. Note that you
cannot _republish_ with this mechanism, so remember to increment the version if
you need to.

Finally, you can publish the whole shebang (unusual) with

    ./gradlew publishAllTemplates

If you don't want to publish your template you can install it locally using the
installTemplate rule.

     ./gradlew installTemplate<TemplateName>

This will install the template to ~/.lazybones/templates so that you can use it without
moving it to bintray first.

And that's it for the project templates.

#### Template versions

You define the version of a template by putting a VERSION file in the root
directory of the template that contains just the version number. For example,
you specify a version of 1.2.8 for the ratpack-lite template by adding the file
src/templates/ratpack-lite/VERSION with the contents

    1.2.8

That's it! The VERSION file will automatically be excluded from the project
template archive.

Contributing templates
----------------------

*For a more comprehensive overview, read the [Template Developers Guide](https://github.com/pledbrook/lazybones/wiki/Template-developers-guide)*

If you have an idea for a project template and want to add it to lazybone's
library, then you have two options:

1. Fork this repo, add your template source to src/templates and submit a pull
   request
2. Keep the source in your own repository, build a zip package for the template,
   publish it to Bintray and finally send a link request to the
   pledbrook/lazybones-templates repository

The second option, a binary contribution, is currently the preferred one.
Otherwise the source for this project could grow too large. Plus it's good for
contributors to take responsibility for publishing their own binaries.

Requirements for a project template:

* Must have a VERSION file in the root directory containing just the current
  version number of the template
* A README, README.txt, README.md (or any README.\* file) in the root of the
  project. This file will be displayed straight after a new project is created
  from the template, so it should give some information about what the template
  contains and how to use it
* An optional lazybones.groovy post install script in the root of the template
  directory (see below for more details). It runs right after the template is
  installed and is deleted after successful completion.
* The name of the binary must be of the form &lt;name>-template-&lt;version>.zip and
  should _not_ contain a parent directory. So a README file must be at the top
  level of the zip.
* The name of the template should ideally be of the form &lt;tool/framework>-&lt;variant>,
  where the variant is optional. For example: ratpack-lite, dropwizard,
  grails-cqrs.

The lazybones.groovy post install script is a generic groovy script with a few extra
helper methods:

* `ask(String message, defaultValue = null)` - asks the user a question and returns their answer, or `defaultValue` if no
answer is provided

* `ask(String message, defaultValue, String propertyName)` - works similarily to the `ask()` above, but allows
grabbing variables from the command line as well based on the `propertyName`.

* `processTemplates(String filePattern, Map substitutionVariables)` - use ant pattern matching to find files and filter their
contents in place using Groovy's `SimpleTemplateEngine`.

* `hasFeature(String featureName)` - checks if the script has access to a feature, `hasFeature("ask")` or
`hasFeature("fileFilter")` would both return true

Here is a very simple example `lazybones.groovy` script that asks the user for
a couple of values and uses those to populate parameters in the template's build
file:

    def params = [:]
    params["groupId"] = ask("What is the group ID for this project?")
    params["version"] = ask("What is the project's initial version?", "0.1", "version")

    processTemplates("*.gradle", params)
    processTemplates("pom.xml", params)

The main Gradle build file might then look like this:

    apply plugin: "groovy"

    <% if (group) { %>group = "${group}"<% } %>
    version = "${version}"

The `${}` expressions are executed as Groovy expressions and they have access
to any variables in the parameter map passed to `processTemplates()`. Scriptlets,
i.e. code inside `<% %>` delimiters, allow for more complex logic.

Alternatively templates can be written using [Mustache templates](http://mustache.github.io/mustache.5.html).
This requires switching the template engine in `lazybones.groovy`:

    import uk.co.cacoethes.lazybones.handlebars.HandlebarsTemplateEngine
    setTemplateEngine(new HandlebarsTemplateEngine())
