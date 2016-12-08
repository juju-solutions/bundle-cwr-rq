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

[jenkins]: [https://jujucharms.com/jenkins/xenial/1]
[cwr]: [https://jujucharms.com/u/kos.tsakalozos/cwr/0]
[review-queue]: [https://jujucharms.com/u/juju-solutions/review-queue/26]
[postgresql]: [https://jujucharms.com/postgresql]
[rabbitmq-server]: [https://jujucharms.com/rabbitmq-server]


# Getting Started

> Note: A bootstrapped Juju controller is required. If Juju is not yet set up,
please follow the [getting-started][] instructions prior to deploying this
bundle.

[getting-started]: https://jujucharms.com/docs/stable/getting-started

Deploy this bundle from the charm store:

    juju deploy cwr-rq

Charms in this bundle provide status messages to indicate their readiness.
Monitor the progress of the deployment with:

    watch juju status

> Note: Once the charms indicate they are ready, use `Ctrl-c` to terminate the
`watch` command and proceed with the following instructions.

Set a password for Jenkins:

    juju config jenkins password=<yourpassword>

CWR needs access to your controller(s) to create models and allocate resources
needed to run charm/bundle tests. Grant this by creating a user on
your bootstrapped controller(s) with appropriate permissions:

    juju add-user ciuser
    juju grant ciuser add-model

To register the controller with the CWR charm, you will need to call the
`register-controller` action and provide a human-friendly name and the
registration token from the above `juju add-user` command.

    juju run-action cwr/0 register-controller token=<controller-name> \
        token=<registration-token>

You should also setup a session with the charm store to allow CWR to release
charms to your namespace. To do this, call the `store-login` action and provide
the base64 representation of an existing auth token. For example:

    charm login
    .........
    export TOKEN=`base64 ~/.local/share/juju/store-usso-token`
    juju run-action cwr/0 store-login charmstore-usso-token="$TOKEN"

At this point, you have the foundation for a powerful charm/bundle CI system.
Workflows that leverage this system are described in the next section.


# Workflows

## Manage the Charm Release Cycle from Github

### Description
Our goal is to build and test a charm/bundle every time code is committed to a
repository. In light of a successful test, the resulting charm/bundle is pushed
to the store and released in the `edge` channel. Similarly, releases to the
`stable` channel can be made by tagging the code in Github when ready.

The rationale of this workflow is that you want charm/bundle updates released as
soon as you are confident that things are working as expected. With good tests,
the CI system can give you that confidence and automatically handle the release
process from a source repo to an `edge` or `stable` channel in the charm store.

### Prerequisites
  * A charm/top charm layer, e.g.: `awesome-charm`
  * Source repository on Github, e.g.: `http://github.com/myself/my-awesome-charm`
  * A charm store namespace, e.g.: `awesome-team`

### Procedure
To include `awesome-charm` in our CI pipeline, we need to call the
`build-on-commit` action:

    juju run-action cwr/0 build-on-commit \
        repo=http://github.com/myself/my-awesome-charm \
        charm-name=awesome-charm \
        push-to-channel=edge \
        lp-id=awesome-team \
        controller=lxd

This will instruct CWR to run a Jenkins job to test `awesome-charm` on your
`lxd` controller and release it to the **edge** channel each time you
**commit** to your repo.

For releasing `awesome-charm` to the stable channel, we need a similar call to
the `build-on-release` action:

    juju run-action cwr/0 build-on-release \
        repo=http://github.com/myself/my-awesome-charm \
        charm-name=awesome-charm \
        push-to-channel=stable \
        lp-id=awesome-team \
        controller=aws

This will instruct CWR to run a Jenkins job to test `awesome-charm` on your
`aws` controller and release it to the **stable** channel any time you
**tag** your source with a release tag.

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

The public review queue is deployed using this bundle and is available for
your organization to duplicate this environment in-house if needed. The
features described above promote collaborative charm/bundle development
coupled with powerful automated test capabilities.

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


# Summary

We have described two workflows that can leverage the charm/bundle CI system
provided by this bundle. Do you have ideas or other workflows built around
CWR? Please let us know by contacting us on the mailing list below.

# Resources

- [Juju mailing list](https://lists.ubuntu.com/mailman/listinfo/juju)
- [Juju community](https://jujucharms.com/community)
