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

Configuration
=============

Changing the default configuration
----------------------------------

If you want to change the default configuration, you can perform any modification you want in the $MY_KARAF_HOME/etc directory.

The context server configuration is kept in the $MY_KARAF_HOME/etc/org.apache.unomi.cluster.cfg . It defines the
addresses where it can be found :

    contextserver.publicAddress=https://localhost:9443
    contextserver.internalAddress=http://127.0.0.1:8181

If you need to specify an Elasticsearch cluster name, or a host and port that are different than the default, 
it is recommended to do this BEFORE you start the server for the first time, or you will loose all the data 
you have stored previously.

To change these settings, you will need to modify a file called 

    $MY_KARAF_HOME/etc/org.apache.unomi.persistence.elasticsearch.cfg

with the following contents:

    cluster.name=contextElasticSearch
    # The elasticSearchAddresses may be a comma seperated list of host names and ports such as
    # hostA:9300,hostB:9300
    # Note: the port number must be repeated for each host.
    elasticSearchAddresses=localhost:9300
    index.name=context
    
Secured events configuration
----------------------------

Unomi secures some events by default. You can find the default configuration in the following file (created after the
first server startup):

    $MY_KARAF_HOME/etc/org.apache.unomi.thirdparty.cfg

Ususally, login events, which operate on profiles and do merge on protected properties, must be secured. For each
trusted third party server, you need to add these 3 lines :

    thirdparty.provider1.key=secret-key
    thirdparty.provider1.ipAddresses=127.0.0.1,::1
    thirdparty.provider1.allowedEvents=login,updateProperties

The events set in allowedEvents will be secured and will only be accepted if the call comes from the specified IP
address, and if the secret-key is passed in the X-Unomi-Peer header.    

Installing the MaxMind GeoIPLite2 IP lookup database
----------------------------------------------------

The Context Server requires an IP database in order to resolve IP addresses to user location.
The GeoLite2 database can be downloaded from MaxMind here :
http://dev.maxmind.com/geoip/geoip2/geolite2/

Simply download the GeoLite2-City.mmdb file into the "etc" directory.

Installing Geonames database
----------------------------

Context server includes a geocoding service based on the geonames database ( http://www.geonames.org/ ). It can be
used to create conditions on countries or cities.

In order to use it, you need to install the Geonames database into . Get the "allCountries.zip" database from here :
http://download.geonames.org/export/dump/

Download it and put it in the "etc" directory, without unzipping it.
Edit $MY_KARAF_HOME/etc/org.apache.unomi.geonames.cfg and set request.geonamesDatabase.forceImport to true, import should start right away.
Otherwise, import should start at the next startup. Import runs in background, but can take about 15 minutes.
At the end, you should have about 4 million entries in the geonames index.
 
REST API Security
-----------------

The Context Server REST API is protected using JAAS authentication and using Basic or Digest HTTP auth.
By default, the login/password for the REST API full administrative access is "karaf/karaf".

The generated package is also configured with a default SSL certificate. You can change it by following these steps :

1. Replace the existing keystore in $MY_KARAF_HOME/etc/keystore by your own certificate :
 
    http://wiki.eclipse.org/Jetty/Howto/Configure_SSL
    
2. Update the keystore and certificate password in $MY_KARAF_HOME/etc/custom.properties file :
 
```
    org.osgi.service.http.secure.enabled = true
    org.ops4j.pax.web.ssl.keystore=${karaf.etc}/keystore
    org.ops4j.pax.web.ssl.password=changeme
    org.ops4j.pax.web.ssl.keypassword=changeme
    org.osgi.service.http.port.secure=9443
```

You should now have SSL setup on Karaf with your certificate, and you can test it by trying to access it on port 9443.

3. Changing the default Karaf password can be done by modifying the etc/users.properties file

Automatic profile merging
-------------------------

The context server is capable of merging profiles based on a common property value. In order to use this, you must
add the MergeProfileOnPropertyAction to a rule (such as a login rule for example), and configure it with the name
 of the property that will be used to identify the profiles to be merged. An example could be the "email" property,
 meaning that if two (or more) profiles are found to have the same value for the "email" property they will be merged
 by this action.
 
Upon merge, the old profiles are marked with a "mergedWith" property that will be used on next profile access to delete
the original profile and replace it with the merged profile (aka "master" profile). Once this is done, all cookie tracking
will use the merged profile.

To test, simply configure the action in the "login" or "facebookLogin" rules and set it up on the "email" property. 
Upon sending one of the events, all matching profiles will be merged.

Securing a production environment
---------------------------------

Before going live with a project, you should *absolutely* read the following section that will help you setup a proper 
secure environment for running your context server.         

Step 1: Install and configure a firewall 

You should setup a firewall around your cluster of context servers and/or Elasticsearch nodes. If you have an 
application-level firewall you should only allow the following connections open to the whole world : 

 - http://localhost:8181/context.js
 - http://localhost:8181/eventcollector

All other ports should not be accessible to the world.

For your Context Server client applications (such as the Jahia CMS), you will need to make the following ports 
accessible : 

    8181 (Context Server HTTP port) 
    9443 (Context Server HTTPS port)
    
The context server actually requires HTTP Basic Auth for access to the Context Server administration REST API, so it is
highly recommended that you design your client applications to use the HTTPS port for accessing the REST API.

The user accounts to access the REST API are actually routed through Karaf's JAAS support, which you may find the
documentation for here : 

 - http://karaf.apache.org/manual/latest/users-guide/security.html
    
The default username/password is 

    karaf/karaf
    
You should really change this default username/password as soon as possible. To do so, simply modify the following
file : 

    $MY_KARAF_HOME/etc/users.properties

For your context servers, and for any standalone Elasticsearch nodes you will need to open the following ports for proper
node-to-node communication : 9200 (Elasticsearch REST API), 9300 (Elasticsearch TCP transport)

Of course any ports listed here are the default ports configured in each server, you may adjust them if needed.

Step 2 : Follow industry recommended best practices for securing Elasticsearch

You may find more valuable recommendations here : 

- https://www.elastic.co/blog/found-elasticsearch-security
- https://www.elastic.co/blog/scripting-security
    
Step 4 : Setup a proxy in front of the context server

As an alternative to an application-level firewall, you could also route all traffic to the context server through
a proxy, and use it to filter any communication.

Integrating with an Apache HTTP web server
------------------------------------------

If you want to setup an Apache HTTP web server in from of Apache Unomi, here is an example configuration using 
mod_proxy.

In your Unomi package directory, in /etc/org.apache.unomi.cluster.cfg for unomi.apache.org
   
   contextserver.publicAddress=https://unomi.apache.org/
   contextserver.internalAddress=http://192.168.1.1:8181
   
and you will also need to change the contextserver.domain in the /etc/org.apache.unomi.web.cfg file

   contextserver.domain=apache.org

Main virtual host config:

    <VirtualHost *:80>
            Include /var/www/vhosts/unomi.apache.org/conf/common.conf
    </VirtualHost>
    
    <IfModule mod_ssl.c>
        <VirtualHost *:443>
            Include /var/www/vhosts/unomi.apache.org/conf/common.conf
    
            SSLEngine on
    
            SSLCertificateFile    /var/www/vhosts/unomi.apache.org/conf/ssl/24d5b9691e96eafa.crt
            SSLCertificateKeyFile /var/www/vhosts/unomi.apache.org/conf/ssl/apache.org.key
            SSLCertificateChainFile /var/www/vhosts/unomi.apache.org/conf/ssl/gd_bundle-g2-g1.crt
    
    
            <FilesMatch "\.(cgi|shtml|phtml|php)$">
                    SSLOptions +StdEnvVars
            </FilesMatch>
            <Directory /usr/lib/cgi-bin>
                    SSLOptions +StdEnvVars
            </Directory>
            BrowserMatch "MSIE [2-6]" \
                    nokeepalive ssl-unclean-shutdown \
                    downgrade-1.0 force-response-1.0
            BrowserMatch "MSIE [17-9]" ssl-unclean-shutdown
    
        </VirtualHost>
    </IfModule>
    
common.conf:

    ServerName unomi.apache.org
    ServerAdmin webmaster@apache.org
    
    DocumentRoot /var/www/vhosts/unomi.apache.org/html
    CustomLog /var/log/apache2/access-unomi.apache.org.log combined
    <Directory />
            Options FollowSymLinks
            AllowOverride None
    </Directory>
    <Directory /var/www/vhosts/unomi.apache.org/html>
            Options FollowSymLinks MultiViews
            AllowOverride None
            Order allow,deny
            allow from all
    </Directory>
    <Location /cxs>
        Order deny,allow
        deny from all
        allow from 88.198.26.2
        allow from www.apache.org
    </Location>
    
    RewriteEngine On
    RewriteCond %{REQUEST_METHOD} ^(TRACE|TRACK)
    RewriteRule .* - [F]
    ProxyPreserveHost On
    ProxyPass /server-status !
    ProxyPass /robots.txt !
    
    RewriteCond %{HTTP_USER_AGENT} Googlebot [OR]
    RewriteCond %{HTTP_USER_AGENT} msnbot [OR]
    RewriteCond %{HTTP_USER_AGENT} Slurp
    RewriteRule ^.* - [F,L]
    
    ProxyPass / http://localhost:8181/ connectiontimeout=20 timeout=300 ttl=120
    ProxyPassReverse / http://localhost:8181/

Changing the default tracking location
--------------------------------------

When performing localhost requests to Apache Unomi, a default location will be used to insert values into the session
to make the location-based personalization still work. You can find the default location settings in the file : 

    org.apache.unomi.plugins.request.cfg
    
that contains the following default settings:

    # The following settings represent the default position that is used for localhost requests
    defaultSessionCountryCode=CH
    defaultSessionCountryName=Switzerland
    defaultSessionCity=Geneva
    defaultSessionAdminSubDiv1=2660645
    defaultSessionAdminSubDiv2=6458783
    defaultSessionIsp=Cablecom
    defaultLatitude=46.1884341
    defaultLongitude=6.1282508

You might want to change these for testing or for demonstration purposes.

Apache Karaf SSH Console
--------------------------------------
The Apache Karaf SSH console is available inside Apache Unomi, but the port has been changed from the default value of
8101 to 8102 to avoid conflicts with other Karaf-based products. So to connect to the SSH console you should use:

    ssh -p 8102 karaf@localhost
    
or the user/password you have setup to protect the system if you have changed it.

ElasticSearch X-Pack Support
--------------------------------------

It is now possible to use X-Pack to connect to ElasticSearch. However, for licensing reasons this is not provided out 
of the box. Here is the procedure to install X-Pack with Apache Unomi:

#Important ! 

Do not start Unomi directly with unomi:start, perform the following steps below first !

#Installation steps

1. Create a directory for all the JARs that you will download, we will call it XPACK_JARS_DIRECTORY
2. Download https://artifacts.elastic.co/maven/org/elasticsearch/client/x-pack-transport/5.6.3/x-pack-transport-5.6.3.jar to XPACK_JARS_DIRECTORY
3. Download https://artifacts.elastic.co/maven/org/elasticsearch/plugin/x-pack-api/5.6.3/x-pack-api-5.6.3.jar to XPACK_JARS_DIRECTORY
4. Download http://central.maven.org/maven2/com/unboundid/unboundid-ldapsdk/3.2.0/unboundid-ldapsdk-3.2.0.jar to XPACK_JARS_DIRECTORY
5. Download http://central.maven.org/maven2/org/bouncycastle/bcpkix-jdk15on/1.55/bcpkix-jdk15on-1.55.jar to XPACK_JARS_DIRECTORY
6. Download http://central.maven.org/maven2/org/bouncycastle/bcprov-jdk15on/1.55/bcprov-jdk15on-1.55.jar to XPACK_JARS_DIRECTORY
7. Download http://central.maven.org/maven2/com/sun/mail/javax.mail/1.5.3/javax.mail-1.5.3.jar to XPACK_JARS_DIRECTORY
8. Edit etc/org.apache.unomi.persistence.elasticsearch.cfg to add the following settings:

        transportClientClassName=org.elasticsearch.xpack.client.PreBuiltXPackTransportClient
        transportClientJarDirectory=XPACK_JARS_DIRECTORY
        transportClientProperties=xpack.security.user=elastic:changeme

    You can setup more properties (for example for SSL/TLS support) by seperating the properties with commas, 
    as in the following example:
    
        transportClientProperties=xpack.security.user=elastic:changeme,xpack.ssl.key=/home/user/elasticsearch-5.6.3/config/x-pack/localhost/localhost.key,xpack.ssl.certificate=/home/user/elasticsearch-5.6.3/config/x-pack/localhost/localhost.crt,xpack.ssl.certificate_authorities=/home/user/elasticsearch-5.6.3/config/x-pack/ca/ca.crt,xpack.security.transport.ssl.enabled=true

9. Launch Karaf and launch unomi using the command from the shell :

        unomi:start
        
Alternatively you could edit the configuration directly from the Karaf shell using the following commands:
        
    config:edit org.apache.unomi.persistence.elasticsearch
    config:property-set transportClientClassName org.elasticsearch.xpack.client.PreBuiltXPackTransportClient
    config:property-set transportClientJarDirectory XPACK_JARS_DIRECTORY
    config:property-set transportClientProperties xpack.security.user=elastic:changeme
    config:update
    unomi:start

You can setup more properties (for example for SSL/TLS support) by seperating the properties with commas, 
as in the following example:
    
    config:property-set transportClientProperties xpack.security.user=elastic:changeme,xpack.ssl.key=/home/user/elasticsearch-5.6.3/config/x-pack/localhost/localhost.key,xpack.ssl.certificate=/home/user/elasticsearch-5.6.3/config/x-pack/localhost/localhost.crt,xpack.ssl.certificate_authorities=/home/user/elasticsearch-5.6.3/config/x-pack/ca/ca.crt,xpack.security.transport.ssl.enabled=true
