<!--
  ~ Licensed to the Apache Software Foundation (ASF) under one or more
  ~ contributor license agreements.  See the NOTICE file distributed with
  ~ this work for additional information regarding copyright ownership.
  ~ The ASF licenses this file to You under the Apache License, Version 2.0
  ~ (the "License"); you may not use this file except in compliance with
  ~ the License.  You may obtain a copy of the License at
  ~
  ~      http://www.apache.org/licenses/LICENSE-2.0
  ~
  ~ Unless required by applicable law or agreed to in writing, software
  ~ distributed under the License is distributed on an "AS IS" BASIS,
  ~ WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  ~ See the License for the specific language governing permissions and
  ~ limitations under the License.
  -->

Login sample
============

This sample is an example of what is involved in integrated a login with Apache Unomi. 

Warning !
---------

The example code uses client-side Javascript code to send the login event. This is only 
done this way for the sake of sample simplicity but if should NEVER BE DONE THIS WAY in real cases.

The login event should always be sent from the server performing the actual login since it must
only be sent if the user has authenticated properly, and only the authentication server can validate this.

Installing the sample
---------------------

1. Login into the Unomi Karaf SSH shell using something like this : 

        ssh -p 8102 karaf@localhost (default password is karaf) 

2. Install the login sample using the following command:

        bundle:install mvn:org.apache.unomi/login-integration-sample/1.2.0-incubating-SNAPSHOT
        
    when the bundle is successfully install you will get an bundle ID back we will call it BUNDLE_ID. 
    
3. You can then do:
    
        bundle:start BUNDLE_ID
        
4. If all went well you can access the login sample HTML page here :

        http://localhost:8181/login/index.html
        
5. You can fill in the form to test it. Note that the hardcoded password is:

        test1234
                