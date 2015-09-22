Gradle Plugin for Node
=======================

[![Build Status](http://goo.gl/gWKi3f)](http://goo.gl/SvKt0Y)
[![Coverage Status](https://coveralls.io/repos/srs/gradle-node-plugin/badge.svg?branch=newstruct&service=github)](https://coveralls.io/github/srs/gradle-node-plugin?branch=newstruct)
[![License](http://goo.gl/MMfZhl)](http://goo.gl/D6iAcM)
[![Download](http://goo.gl/7HBa4S)](http://goo.gl/FfQphx)

This plugin enables you to run any NodeJS script as part of your build. It does
not depend on NodeJS (or NPM) being installed on your system. The plugin will
download and manage NodeJS distributions, unpack them into your local `.gradle`
directory and use them from there.

Installing the Plugin
---------------------

Releases of this plugin are hosted at [Bintray][] and is part of the [jCenter
repository][]. Development builds are published for every commit to the master
branch. These `SNAPSHOT`s are hosted on the [OJO repository][] and to use them
you will need to add OJO to your `buildscript` configuration.

[Bintray]: http://bintray.com
[OJO repository]: http://oss.jfrog.org
[jCenter repository]: https://bintray.com/bintray/jcenter

Setup the plugin like this:

    plugins {
      id "com.moowork.node" version "0.10"
    }

Or using the old (pre 2.1) way:

    buildscript {
      repositories {
        jcenter()
        // If you want to use a SNAPSHOT build, add the OJO repository:
        maven {
          name 'JFrog OSS snapshot repo'
          url  'https://oss.jfrog.org/oss-snapshot-local/'
        }
      }
      dependencies {
        classpath 'com.moowork.gradle:gradle-node-plugin:0.10'
      }
    }

Include the plugin in your build.gradle file like this:

    apply plugin: 'com.moowork.node'

Running a NodeJS Script
-----------------------

To use this plugin you have to define some tasks in your build.gradle file. If
you have a NodeJS script in `src/scripts/my.js`, then you can execute this by
defining the following Gradle task:

    task myScript(type: NodeTask) {
      script = file('src/scripts/my.js')
    }

You can also add arguments, like this:

    task myScript(type: NodeTask) {
      script = file('src/scripts/my.js')
      args = ['arg1', 'arg2']
    }

`NodeTask` is a wrapper around the core `Exec` task. You can set the
`ignoreExitValue` property on it:

    task myScript(type: NodeTask) {
      script = file('src/scripts/my.js')
      ignoreExitValue = true
    }

You can also customize all other values on the `ExecSpec` by passing a closure
to `execOverrides`. It's executed last, possibly overriding already set
parameters.

    task myScript(type: NodeTask) {
      script = file('src/scripts/my.js')
      execOverrides {
        it.ignoreExitValue = true
        it.workingDir = 'somewhere'
        it.standardOutput = new FileOutputStream('logs/my.log')
      }
    }

When executing this for the first time, it will run a nodeSetup task that
downloades NodeJS (for your platform) and NPM (Node Package Manager) if on
windows (other platforms include it into the distribution).

Executing `npm` Tasks
---------------------

When adding the node plugin, you will have a npmInstall task already added. This
task will execute `npm install` and installs all dependencies in `package.json`.
It will only run when changes are made to `package.json` or `node_modules`.
Execute it like this:

    $ gradle npmInstall

All npm command can also be invoked using underscore notation based on a gradle
rule:

    $ gradle npm_install
    $ gradle npm_update
    $ gradle npm_list
    $ gradle npm_cache_clean
    ...

These however are not shown when running gradle tasks, as they generated
dynamically. However they can be used for dependency declarations, such as:

    npm_install.dependsOn(npm_cache_clean)

More arguments can be passed via the build.gradle file:

    npm_update {
      args = ['--production', '--loglevel', 'warn']
    }

If you want to extend the tasks more or create custom variants, you can extend
the class `NpmTask`:

    task installExpress(type: NpmTask) {
      // install the express package only
      args = ['install', 'express', '--save-dev']
    }

Configuring the Plugin
----------------------

You can configure the plugin using the "node" extension block, like this:

    node {
      // Version of node to use.
      version = '0.11.10'

      // Version of npm to use.
      npmVersion = '2.1.5'

      // Base URL for fetching node distributions (change if you have a mirror).
      distBaseUrl = 'http://nodejs.org/dist'

      // If true, it will download node using above parameters.
      // If false, it will try to use globally installed node.
      download = true

      // Set the work directory for unpacking node
      workDir = file("${project.buildDir}/nodejs")

      // Set the work directory where node_modules should be located
      nodeModulesDir = file("${project.projectDir}")
    }

**Note** that `download` flag is default to `false`. This will change in future versions.

Using a Custom (project-local) Version of `npm`
-----------------------------------------------

The plugin will use a locally-installed `npm` if it exists, regardless of the
method of installation.

If you would like the plugin to install use a custom version of npm rather than
the one bundled with the version of node installation, you can set `npmVersion`
in the `node` extension block. The plugin will install the npm to the project's
`node_modules` directory by configuring the `npmSetup` task.

Building the Plugin
-------------------

To build the plugin, just type the following command:

    ./gradlew clean build
