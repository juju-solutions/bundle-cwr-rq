<!--
  Licensed to the Apache Software Foundation (ASF) under one or more
  contributor license agreements.  See the NOTICE file distributed with
  this work for additional information regarding copyright ownership.
  The ASF licenses this file to You under the Apache License, Version 2.0
  (the "License"); you may not use this file except in compliance with
  the License.  You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License.
-->
# Overview

This bundle deploys a reference infrastructure for testing, reviewing and releasing charms and bundles to the Juju Store. The CI used in this bundle is Jenkins paired with a set of charm specific tools, namely:

  * [The Review Queue][]
  * [Cloud Weather Report][]
  * [Bundletester][]
  * [Matrix][]

[The Review Queue]: https://github.com/juju-solutions/review-queue
[Cloud Weather Report]: https://github.com/juju-solutions/cloud-weather-report
[Bundletester]: https://github.com/juju-solutions/bundletester
[Matrix]: https://github.com/juju-solutions/matrix


## Bundle Composition

The applications that comprise this bundle are spread across 2 machines as follows:

  * Machine 1
    * Jenkins
    * Juju CI environment
  * Machine 2
    * The review queue
    * Postresql
    * Rabbit MQ

Deploying this bundle sets the foundation for a charm oriented CI infrastructure. Additional configuration steps are required to reach a fully functional CI pipeline tailored to your specific needs. In particular you will need to configure the `juju-ci-env` application to be able to interact with the juju store and also to register with juju controllers. The latter are used to draw resources from and run the charm and bundle tests.

Please, find additional configuration steps in the README documentation of the respective charms:

  * [jenkins][],
  * [review-queue][] and
  * [juju-ci-env][]

[jenkins]: [https://jujucharms.com/jenkins/xenial/1]
[review-queue]: [https://jujucharms.com/u/juju-solutions/review-queue/26]
[juju-ci-env]: [https://jujucharms.com/u/kos.tsakalozos/juju-ci-env/4]


# Getting Started

## Basic Configuration

As soon as the bundle is deployed you will need to setup a password for Jenkins:

    juju config jenkins password=<yourpassword>

To let the CI know about the existence of a controller you will need to login to Jenkins and trigger the `RegisterCOntroller` job. This job requires you to provide the controller registration token and a human friendly name. To aquire a registration token you would need to add a user to the controller you have already bootstrapped and grant the appropriate permissions.

    juju add-user ciuser
    juju grant ciuser add-model

While logged in to Jenkins you should also setup a session with the Juju Store. To do so you should trigger the `InitJujuStoreSession` job providing your (build-bot's) credentials


## Manage the Release Cycle of your Charm

Prerequisites:
  * A charm/top charm layer - named: awesome-charm
  * Source on Github - repo: http://github.com/myself/my-awesome-charm
  * A namespace on Juju store - Launchpad ID: mycorp

Our goal here is to build and test our charm every time we commit code to the head of the master branch of the repo. In light of a successful test, the resulting charm is pushed to the store and released under the edge channel. Releases to the stable channel are managed through github by adding a release tag. Upon a the creation of a tag the CI automatically performs the build, test, release cycle.

The rational of the above workflow is that you always want to test your charm and as soon as you are confident you just tag the code on githab and release it.

To build our awesome-charm we need to call the `buildcharmoncommit` action:

    juju run-action juju-ci-env/0 buildcharmoncommit gitrepo=http://github.com/myself/my-awesome-charm charmname=awesome-charm  pushtochannel=edge lpid=mycorp controller=lxd

The above action will add a job into jenkins and use the `lxd` controller.

For releasing our charm to the stable channel we need a similar call to the `buildcharmonrelease` action:

    juju run-action juju-ci-env/0 buildcharmonrelease gitrepo=http://github.com/myself/my-awesome-charm charmname=awesome-charm  pushtochannel=stable lpid=mycorp controller=aws

For completeness the releases are tested on AWS.

What we describe here is just one of the workflows you can have to release your charm, please let us know about your workflow in the list below.

# Resources

- [Juju mailing list](https://lists.ubuntu.com/mailman/listinfo/juju)
- [Juju community](https://jujucharms.com/community)
