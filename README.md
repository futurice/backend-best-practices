Backend development best practices
==================================

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [N Commandments](#n-commandments)
- [General points on guidelines](#general-points-on-guidelines)
- [Development environment setup in README.md](#development-environment-setup-in-readmemd)
- [Data persistence](#data-persistence)
- [Environments](#environments)
  - [Local development environment](#local-development-environment)
  - [Continuous integration environment](#continuous-integration-environment)
  - [Testing environment](#testing-environment)
  - [Staging environment](#staging-environment)
  - [Production environment](#production-environment)
- [Bill of Materials](#bill-of-materials)
- [Security](#security)
  - [Credentials](#credentials)
  - [Secrets](#secrets)
  - [Login Throttling](#login-throttling)
  - [User Password Storage](#user-password-storage)
  - [Audit Log](#audit-log)
  - [Suspicious Action Throttling and/or blocking](#suspicious-action-throttling-andor-blocking)
  - [Anonymized Data](#anonymized-data)
  - [Temporary file storage](#temporary-file-storage)
  - [Dedicated vs Shared server environment](#dedicated-vs-shared-server-environment)
- [Release checklist](#release-checklist)
- [General questions to consider](#general-questions-to-consider)
- [Generally proven useful tools](#generally-proven-useful-tools)
- [License](#license)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# N Commandments

1. README.md in the root of the repo is the docs
2. Single command run
3. Single command deploy
4. Repeatable and re-creatable builds
5. Build artifacts bundle a "Bill of Materials"
6. Use [UTC as the timezone](http://yellerapp.com/posts/2015-01-12-the-worst-server-setup-you-can-make.html) all around

# General points on guidelines

We do not want to limit ourselves to certain tech stacks or frameworks. Different problems require different solutions, and hence these guidelines are valid for various backend architectures.

# Development environment setup in README.md

Document all the parts of the development/server environment. Strive to use the same setup and versions on all environments, starting from developer laptops, and ending with the actual production environment. This includes database, application server, proxy server (nginx, apache, ...), SDK version(s), gems/libraries/modules.

Automate the setup process as much as possible. For example, [Docker Compose](https://docs.docker.com/compose/) could be used in both production and development to set up a complete environment, where [Dockerfiles](https://docs.docker.com/articles/dockerfile_best-practices/) fetch all parts of the software, and contain the necessary scripting to setup the environment and all the parts of it. Consider using archived copies of installers, in case upstream packages later become unavailable. A minimum precaution is to keep SHA-1 checksums of the packages, and to make sure that the checksum matches when the packages are installed.

Consider storing any relevant parts of the development environment and dependencies in some persistent storage. If the environment can be built using Docker, one possible way to do this is to use [docker export](http://docs.docker.com/reference/commandline/cli/#export).

# Data persistence

## General considerations

Whichever persistence solution your project uses, there are general considerations that you should follow:

* Have backups that are verified to work
* Have scripts or other tooling for copying persistent data from one env to another, e.g. from prod to staging in order to debug something
* Have plans in place for rolling out updates to persistence solution (e.g. database server security update)
* Have plans in place for scaling up persistence solution
* Have monitoring in place to verify health of persistence solution

## Persistence solutions

### RDBMS

### NoSQL

#### Document storage

#### Key-value store

#### Graph database

### Blob storage

# Environments

This section describes the environments you should have, at a minimum. It might sound like a lot, [but there is a purpose for each one](http://futurice.com/blog/five-environments-you-cannot-develop-without).

- [Local development](#local-development-environment)
- [Continuous integration](#continuous-integration-environment)
- [Testing](#testing-environment)
- [Staging](#staging-environment)
- [Production](#production-environment)

## Local development environment

This is your local development environment. You probably should not have a shared external development environment, instead you should work to make it possible to run the entire system locally, by stubbing or mocking third-party services as needed.

## Continuous integration environment

CI is (among other things) for making sure that your software builds and automated tests pass after every change.

## Testing environment

This is a shared environment that code is deployed to as often as possible, preferably every time code is committed to the mainline branch. It can be broken from time to time, especially in the active development phase. It is an important canary environment and is as similar to production as possible. Any external integrations are set up to use staging-level versions of other services.

## Staging environment

Staging is set up exactly like production. No changes to the production environment happen before having been rehearsed here first. Any mysterious production issues can be debugged here.

## Production environment

The big iron. Logged, monitored, cleaned up periodically, squared away and secured.

# Bill of Materials

This document must be included in every build artifact and shall contain the following:

1. What version(s) of an SDK and critical tools were used to produce it
1. Which dependencies have been included
1. A globally unique revision number of the build (i.e. a git SHA-1 hash)
1. Environment and variables used when building the package
1. List of failed tests or checks


# Security

Be aware of possible security threats and problems. You should at least be familiar with the [OWASP Top 10 vulnerabilities](https://www.owasp.org/index.php/Category:OWASP_Top_Ten_Project), and you should of monitor vulnerabilities in any third party software you use.

The following are good generic security guidelines:

## Credentials

Never send credentials unencrypted over public network. Use always encryption (such as HTTPS, SSL, etc.).

## Secrets

Never store secrets (passwords, keys, etc.) in the sources in version control! It is very easy to forget they are there and the project source tends to end up in many places (developer machines, development test servers, etc) which unnecessarily increases the risk of an important secret being compromised. Also, version control has the nasty feature of overwriting file permissions, so even if you secure your config file permissions, the next time you check out the source, the permissions would be overwritten to the default public-readable.

Probably the easiest way to handle secrets is to put them in a separate file on the servers that need them, which is ignored from version control. You can keep in version control e.g. a `.sample` file with fake values to illustrate what should go in the real file. In some cases, it is not easy to include a separate configuration file from the main configuration: if this happens, consider using environment variables, or writing the config file from a version-controlled template on deployment.

## Login Throttling

Place limits on the amount of login attempts allowed per client per unit of time. Lock a user account for specific time after a given number of failed attempts (e.g. lock for 5 minutes after 20 failed login attempts).
The aim of these measures is make online brute-force attacks against usernames/passwords infeasible.

## User Password Storage

> Never EVER store passwords in plaintext!

Never store passwords in reversible encrypted form, unless absolutely required by the application / system. Here is a good article about what and what not to do: https://crackstation.net/hashing-security.htm

If you do need to be able to obtain plaintext passwords from the database, here are some suggestions that you can follow:
If passwords won't be converted back to plaintext often (e.g. special procedure is required), keep decryption keys away from the application that accesses the database regularly.

If passwords still need to be regularly decrypted, separate the decryption functionality from the main application as much as possible - e.g. separate server accepts requests to decrypt a password, but enforces higher level of control - throttling, authorization, etc.

Whenever possible (it should be possible in great majority of cases), store passwords using a good one-way hash with good random salt. And, no, SHA1 is not a good choice for a hashing function in this context. Hash functions that are designed with passwords in mind are deliberately slower, which makes offline brute-force attacks more time consuming, hence less feasible. See this post: http://security.stackexchange.com/questions/211/how-to-securely-hash-passwords/31846#31846

## Audit Log

For applications handling sensitive data, especially where certain users are allowed relatively wide access or control, it's good to maintain some kind of audit logging - storing a sequence of actions / events that took place in the system, together with the event/source originator (user, automation job, etc). This can be, e.g:

    2012-09-13 03:00:05 Job "daily_job" performed action "delete old items".
    2012-09-13 12:47:23 User "admin_user" performed action "delete item 123".
    2012-09-13 12:48:12 User "admin_user" performed action "change password of user foobar".
    2012-09-13 13:02:11 User "sneaky_user" performed action "view confidential page 567".
    ...

The log may be simple text file or store in a database. At least three items are good to have: exact timestamp, action/event originator (who did this), action/event (what was done). Logged actions, of course depend on what is important for the application itself.

The audit log may be part of the normal application log, but the emphasis here is on logging who did what and not only that a certain action was performed. If possible, the audit log should be made tamper-proof, e.g. only be accessible by a dedicated logging proces/user and directly by the application.

## Suspicious Action Throttling and/or blocking

This can be seen as a generalization of the Login Throttling, this time introducing similar mechanics for arbitrary actions that are deemed "suspicious" within the context of the application. For example, an ERP system which allows normal users access to a substantial amount of information, but expects users to be concerned only with a small subset of that information, may limit attempts to access larger than expected datasets too quickly. E.g. prevent users from downloading list of all customers, if users are supposed to work on one or two customers at a time. Note that this is different from limiting access completely - users are still allowed to retrieve information about any customer, just not all of them at once. Depending on the system, throttling might not be enough - e.g. when one invokes an action on all resources with a single request. Then blocking might be required. Note the difference between making 1000 requests in 10 seconds to retrieve full customer information, one customer at a time, and making a single request to retrieve that information at once.

What is suspicious here depends strongly on the expected use of the application. E.g. in one system, deleting 10000 records might be completely legitimate action, while in another - not.

## Anonymized Data

Whenever large datasets are exported to third parties, data should be anonymized as much as possible, given the intended use of the data. For example, if the third party service will provide general statistical analysis on a customer database, it probably does not need to know the names, addresses or other personal information for individual customers. Even a generic customer ID number might be too revealing, depending on the data set. Take a look at this article: http://arstechnica.com/tech-policy/2009/09/your-secrets-live-online-in-databases-of-ruin/.

## Temporary file storage

Make sure you are aware where your application is storing temporary files. If you are using publicly accessible directories (which are most probably the default) like `/tmp` and `/var/tmp`, make sure you create your files with mode 600, so that they are readable only by the user, your application is running as. Alternatively, have a protected directory for storing temporary files (directory accessible only by the application user).

## Dedicated vs Shared server environment

The security threats can be quite different depending on whether the application is going to run in a shared or dedicated environment. Shared here means that there are other (not necessarily 3rd party)  applications running on the same server. In that case, having appropriate file permissions becomes critical, otherwise application source code, data files, temporary files, logs, etc might end up accessible by unintended users. Then a security breach in a 3rd party application might result in your application being compromised.

You can never be sure what kind of environment your application will run for its entire life time - it may start on a dedicated server, but as time goes, 3rd party applications might be added to the same system. That is why it is best to plan from the very first moment, that your application runs in a shared environment, and take all precautions. Here's a list (not exhaustive) of files/directories you need to think about:

* application source code
* data directories
* temporary storage directories (often by default the system wide /tmp might be used - see above)
* configuration files
* version control directories - .git, .hg, .svn, etc.
* startup scripts (may contain initialization variables, secrets, etc)
* log files
* crash dumps
* private keys (SSL, SSH, etc)
* etc.

Sometimes, some files need to be accessible by different users (e.g. static content served by apache). In that case, take care to allow only access to what is really needed.

Keep in mind that on UNIX/Linux filesystem, write access to a directory is a very powerful permission - it allows you to delete files in that directory and recreate them (which results in modified file). /tmp and /var/tmp are by default safe from this effect, because of the sticky bit that should be set on those.

Additionally, as mentioned in the secrets section, file permissions are not preserved in version control, so even if you set them once, the next checkout/update/whatever may override them. Good idea is then to have a Makefile, script, a version control hook, etc that would set the correct permissions when updating the source.

# Release checklist

When you are ready to make a release, remember to check off everything on your release checklist! The resulting peace of mind, repeatability and dependability is a great boon.

You *do* have one, right? If you don't, here is a good generic starting point for you:

* [ ] Deploying works the same no matter which environment you are deploying to
* [ ] All environments have well defined names, and they are referred to using those names
* [ ] All environments have the same underlying software stack
* [ ] All environment configuration is version controlled (web server config, CI build scripts etc.)
* [ ] The product has been tested from the networks from where it will be used (e.g. public Internet, customer LAN)
* [ ] The product has been tested with all of the targeted devices
* [ ] There is a simple way to find out what code is running in any given environment
* [ ] A versioning scheme has been defined
* [ ] Any version of the product should be easily mappable to a state of the code base
* [ ] Rolling back a deployment is possible
* [ ] Backups are running
* [ ] Restoring from a backup has been tested
* [ ] No secrets are stored in version control
* [ ] Logging is turned on
* [ ] There is a well defined process for accessing and searching through logs
* [ ] Logging includes exceptions and stack traces where appropriate
* [ ] Errors can be mapped to stack traces
* [ ] Release notes have been written
* [ ] Server environments are up-to-date
* [ ] A plan to update server environments exists
* [ ] The product has been load tested
* [ ] A method exists for replicating the state of one environment in another (e.g. copy prod to QA to reproduce an error)
* [ ] All repeating release processes have been automated

# General questions to consider

* What is the expected/required life-span of the project?
* Is the project one-off, or will there be continuous development?
* What is the release cycle for a version of the service
* What environments (dev, test, staging, prod, ...) are going to be set up?
* What is the maximum allowed downtime for the production service?
* How mature is the technology? Is major changes that break backward compatibility to be expected?

# Generally proven useful tools

* [HTTPie](https://github.com/jakubroztocil/httpie) is a great tool for testing APIs on the command line. It's simple to pass in custom headers and cookies, and it even has session support.
* [jq](http://stedolan.github.io/jq/) is a CLI JSON processor. Massage JSON data coming in from cURL (or of course HTTPie!) at will. Another great tool for API testing or exploration.

# License

[Futurice Oy](http://www.futurice.com)
Creative Commons Attribution 4.0 International (CC BY 4.0)
