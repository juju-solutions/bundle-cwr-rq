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

This bundle deploys a reference infrastructure for testing, reviewing and releasing charms and bundles
to the Juju Store. The CI used in this bundle is Jenkins paired with a set of charm specific tools, namely:

  * [The Review Queue][]
  * [Cloud Weather Report][]
  * [Bundletester][]
  * [Matrix][]

[The Review Queue]: https://github.com/juju-solutions/review-queue
[Cloud Weather Report]: https://github.com/juju-solutions/cloud-weather-report
[Bundletester]: https://github.com/juju-solutions/bundletester
[Matrix]: https://github.com/juju-solutions/matrix


## Bundle Composition

The applications that comprise this bundle are spread across 2 machines as
follows:

  * Machine 1
    * Jenkins
    * Juju CI environment
  * Machine 2
    * The review queue
    * Postresql
    * Rabbit MQ

Deploying this bundle sets the foundation for a charm oriented CI infrastructure.
Additional configuration steps are required to reach a fully functional
CI pipeline tailored to your specific needs.
In particular you will need to configure the `juju-ci-env` application to be able to
interact with the juju store and also to register with juju controllers.
The latter are used to draw resources from and run the charm and bundle tests.

Please, find additional configuration steps in the README documentation of
the respective charms:

  * [jenkins][],
  * [review-queue][] and
  * [juju-ci-env][]

[jenkins]: [https://jujucharms.com/jenkins/xenial/1]
[review-queue]: [https://jujucharms.com/u/juju-solutions/review-queue/26]
[juju-ci-env]: [https://jujucharms.com/u/kos.tsakalozos/juju-ci-env/4]


# Resources

- [Juju mailing list](https://lists.ubuntu.com/mailman/listinfo/juju)
- [Juju community](https://jujucharms.com/community)
