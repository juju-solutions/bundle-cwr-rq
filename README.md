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

This bundle deploys a reference platform for reviewing, testing, and releasing
Juju Charms and Bundles into the Juju Charm Store. The CI used in this bundle
is Jenkins, paired with the following charm-specific tools:

  * [Cloud Weather Report (CWR)][]
  * [Bundletester][]
  * [Matrix][]
  * [Review Queue][]

[Cloud Weather Report (CWR)]: https://github.com/juju-solutions/cloud-weather-report
[Bundletester]: https://github.com/juju-solutions/bundletester
[Matrix]: https://github.com/juju-solutions/matrix
[Review Queue]: https://github.com/juju-solutions/review-queue

The cornerstone of this bundle is Cloud Weather Report (CWR). The `cwr` charm
handles CI requests (either from a webhook or the Review Queue), manages
the necessary models on your controller(s), dispatches jobs to Jenkins,
provides job status to the requester, and can automatically release charms to
your namespace in the charm store.

> Note: We have a variation of this bundle called [cwr-ci][] that includes all
of the same components without the Review Queue. If you are interested in a
charm/bundle CI system but do not need your own source review application, we
recommend you have a look at [cwr-ci][].

[cwr-ci]: https://github.com/juju-solutions/bundle-cwr-ci

## Bundle Composition
The charms that comprise this bundle are spread across 4 machines. Additional
information about these charms can by found in the following linked READMEs:

  * [jenkins][]
  * [cwr][] (colocated on the jenkins machine)
  * [review-queue][]
  * [postgresql][]
  * [rabbitmq-server][]

[jenkins]: https://jujucharms.com/jenkins/xenial
[cwr]: https://jujucharms.com/u/kos.tsakalozos/cwr
[review-queue]: https://jujucharms.com/u/juju-solutions/review-queue
[postgresql]: https://jujucharms.com/postgresql
[rabbitmq-server]: https://jujucharms.com/rabbitmq-server


# Getting Started

> Note: A bootstrapped Juju controller is required. If Juju is not yet set up,
please follow the [getting-started][] instructions prior to deploying this
bundle.

[getting-started]: https://jujucharms.com/docs/stable/getting-started

Deploy this bundle from the charm store:

    juju deploy cs:~juju-solutions/cwr-rq

Charms in this bundle provide status messages to indicate their readiness.
Monitor the progress of the deployment with:

    watch -c juju status --color

> Note: Once the charms indicate they are ready, use `Ctrl-c` to terminate the
`watch` command and proceed with the following instructions.

At this point, the `cwr` charm needs access to your controller(s) to create
models and allocate resources needed to run charm/bundle tests. The steps
required to do this are covered in detail in the *Getting Started* section of
the [cwr-ci bundle readme][cwr-ci bundle].

Those details will not be repeated here, but a summary of the procedure is as
follows:

* Set a jenkins admin password
* Add a user to your controller(s)
* Grant `add-model` permissions to the new user(s)
* Run the `register-controller` action
* Run the `set-credentials` action
* Optionally run the `store-login` action

[cwr-ci bundle]: https://jujucharms.com/u/juju-solutions/cwr-ci


# Workflows

The `cwr` test and reporting capabilities of this bundle are covered in detail
in the *Workflows* and *Job Status* sections of the
[cwr-ci bundle readme][cwr-ci bundle]. As before, the overlapping documentation
will not be repeated here, but at a glance, the following usage scenarios are
supported via `cwr` charm actions:

* Test a charm when a commit is made to a charm source repository
* Test a charm when a [release][gh-release] is created in a Github charm source
repository
* Test a charm when a pull request is created in a Github charm source
repository
* Test a bundle when an included charm is updated in the Charm Store

[gh-release]: https://help.github.com/articles/creating-releases/

The Review Queue workflow is unique to this bundle, and is described in detail
in the following section.

## Ensure Charm Quality with the Review Queue

### Description
The Review Queue is a web app for reviewing charm/bundle source. It is typically
used to gate inclusion into the recommended section (i.e. top-level namespace)
of the charm store. The public review queue is available for anyone to
make submissions to the charm store at [review.jujucharms.com][].

Upon requesting a review of your namespace charm
(e.g. `~awesome-team/awesome-charm`), the review queue will invoke CWR to test
your charm on all known substrates and return those results. In addition, the
review queue provides a user interface for reviewing source that includes
a checklist of requirements that must be met to become a recommended
charm/bundle in the store.

[review.jujucharms.com]: https://review.jujucharms.com

The public review queue will soon be deployed using this bundle and is
available for your organization to duplicate this environment in-house if
needed. The features described above promote collaborative charm/bundle
development coupled with powerful automated test capabilities.

### Prerequisites
You'll need an `https` URL to map to the Review Queue IP address. Review queue
authentication is handled by Ubuntu One SSO, which requires a secure callback
URL. If you cannot allocate an `https` URL for the IP address of the review
queue machine, we recommend using [ngrok][]. This free service will forward
requests from their provisioned `https` URL to/from the review queue IP address.

[ngrok]: https://ngrok.com/

The IP address of the review queue can be found in the `Public address` field
of `juju status` output:

    juju status review-queue


### Procedure
To access the Review Queue interface, first expose the review queue
application:

    juju expose review-queue

Now configure the `base_url` with your `https` URL:

    juju config review-queue base_url=https://<your-url>

The web interface will be available at `https://<your-url>`. Login to
the application and click `Request a Review`. Submit a charm/bundle from your
namespace that you would like to review (e.g. `~awesome-team/awesome-charm`).

The review queue will automatically submit your charm to CWR for testing and
will display results when they become available. The code inspection and charm
store policy checklist are also available to review your source code.


# Resources

## Community

- `#juju` on Freenode
- [Juju mailing list](https://lists.ubuntu.com/mailman/listinfo/juju)
- [Juju community](https://jujucharms.com/community)

## Technical

The CWR/RQ system leverages a plethora of tooling from the Juju ecosystem.
Details can be found at the following project links:

- [cloud-weather-report](https://github.com/juju-solutions/cloud-weather-report)
- [bundletester](https://github.com/juju-solutions/bundletester)
- [matrix](https://github.com/juju-solutions/matrix)
- [python-libjuju](https://github.com/juju/python-libjuju)
- [review-queue](https://github.com/juju-solutions/review-queue)
