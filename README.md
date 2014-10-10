Backend development best practices
==================================

# N Commandments
1. README.md in the root of the repo is the docs
1. Single command run
1. Single command deploy
1. Repeatable and re-creatable builds
1. Build artifacts bundle a "Bill of Materials"


# General 
We do not want to limit ourselves to certain tech stacks or frameworks. Differnent 
problems require different solutions, and hence these guidelines should be valid 
for various backend architectures. 

## Development environment setup in README.md

Document all the parts of the development/server environment. Strive to use the same
setup and versions on all environments, starting from developer laptops, and ending 
with the actual production environment. This includes database, application server, 
proxy server (nginx, apache, ...), SDK version(s), gems/libraries/modules. 

Automate the setup process as much as possible. E.g. a docker file could fetch all 
parts of the software, and contain the necessary scripting to setup the environment and 
all the parts of it. Local/Cloud copies of installers can and should be used, if there 
is any risk of the packages to not be available at a later stage. A minimum precaution 
is to keep SHA-1 checksums of the packages, and to make sure that the checksum matches 
when the packages are installed. 

Consider storing any relevant parts of the development environment and dependencies in 
some persistent storage. If the environment can be built using docker, one possible 
way to do this is to use [docker export](http://docs.docker.com/reference/commandline/cli/#export).

## Data persistence

Under construction: How do we handle persisted data between versions, and between 
different environments

Shall this contain discussion on database stuff, or also any generated files?


## Environments

Under construction: List of possible environments, and short description
- Build/CI
- Dev
- QA
- PerformanceTest
- Staging
- Production



#### Bill of Materials
This document must be included in every build artifact and shall contain the following: 

1. What version(s) of an SDK and critical tools were used to produce it
1. Which dependencies have been included 
1. A globally unique revision number of the build (i.e. a git SHA-1 hash)
1. Environment and variables used when building the package
1. List of failed tests or checks


## Security

Be aware of possible security threats and problems. Good places of information include

* [FutuHosting Security Guidelines](https://confluence.futurice.com/display/FutuHosting/Security+Guidelines)
* [OWASP Top 10 vulnerabilities 2013](https://www.owasp.org/index.php/Top_10_2013)

## General questions to consider 

* What is the expected/required life-span of the project?
* Is the project one-off, or will there be continuous development? 
* What is the release cycle for a version of the service
* What environments (dev, test, staging, prod, ...) are going to be set up? 
* What is the maximum allowed downtime for the production service? 
* How mature is the technology? Is major changes that break backward compatibility to be expected?

