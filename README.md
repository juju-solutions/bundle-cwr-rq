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

This bundle deploys the Juju CI solution.


# Instructions

After the bundle is deployed you need to set a password for the jenkins admin user.

    juju config jenkins password=<your_password>

You will also need to give the CI access to a controller. To do so you would need to
issue the following on a controller you have already bootstraped:

    juju add-user ciuser
    juju grant ciuser superuser

Then login into jenkins (user admin) and build the InitCwr job providing the token you got form
the add-user commnad and a user friendly name for the controller.
While you are logged in you can initialise the session between jenkins and the Juju Store
so that you can push the build artifacts to the store. 
<!-- we could have an action for these -->


# Resources

- [Juju mailing list](https://lists.ubuntu.com/mailman/listinfo/juju)
- [Juju community](https://jujucharms.com/community)
