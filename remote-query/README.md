remote-query: Use JDG remotely through Hotrod
================================================
Author: Adrian Nistor
Level: Intermediate
Technologies: Infinispan, Hot Rod, Remote Query
Summary: Demonstrates how to query Infinispan remotely using the Hot Rod client.
Target Product: JDG
Product Versions: JDG 6.2
Source: <https://github.com/infinispan/jdg-quickstart>

What is it?
-----------

Hot Rod is a binary TCP client-server protocol used in JBoss Data Grid. The Hot Rod protocol facilitates faster client and server interactions in comparison to other text based protocols and allows clients to make decisions about load balancing, failover and data location operations.

This quickstart demonstrates how to connect remotely to JBoss Data Grid (JDG) to store, retrieve, remove and query data from cache using the Hot Rod protocol. It is a simple address book manager console application that allows you to create, edit and remove Persons, manage a list of phone numbers for each Person, query and print the contents of the data grid, all using the Hot Rod based connector.


System requirements
-------------------

All you need to build this project is Java 6.0 (Java SDK 1.6) or better, Maven 3.0 or better.

The application this project produces is designed to be run on JBoss Data Grid 6.2

 
Configure Maven
---------------

If you have not yet done so, you must [Configure Maven](../../README.md#configure-maven) before testing the quickstarts.


Configure JDG
-------------

<!-- TODO JBOSS_HOME vs. JDG_HOME. which one do we need to set? -->

1. Obtain JDG server distribution on Red Hat's Customer Portal at https://access.redhat.com/jbossnetwork/restricted/listSoftware.html

2. Install a JDBC driver into JDG (since JDG includes H2 by default, this step may be skipped for the scope of this example). More information can be found in the DataSource Management chapter of the Administration and Configuration Guide for JBoss Enterprise Application Platform on the Customer Portal at <https://access.redhat.com/site/documentation/JBoss_Enterprise_Application_Platform/> . _NOTE: JDG does not support deploying applications so one cannot install it as a deployment._

3. This Quickstart uses JDBC to store the cache. To permit this, it's necessary to alter JDG configuration file (`JDG_HOME/standalone/configuration/standalone.xml`) to contain the following definitions:
   
* Datasource subsystem definition:

    
        <subsystem xmlns="urn:jboss:domain:datasources:1.1">
            <!-- Define this Datasource with jndi name  java:jboss/datasources/ExampleDS -->
            <datasources>
                <datasource jndi-name="java:jboss/datasources/ExampleDS" pool-name="ExampleDS" enabled="true" use-java-context="true">
                    <!-- The connection URL uses H2 Database Engine with in-memory database called test -->
                    <connection-url>jdbc:h2:mem:test;DB_CLOSE_DELAY=-1</connection-url>
                    <!-- JDBC driver name -->
                    <driver>h2</driver>
                    <!-- Credentials -->
                    <security>
                        <user-name>sa</user-name>
                        <password>sa</password>
                    </security>
                </datasource>
                <!-- Define the JDBC driver called 'h2' -->
                <drivers>
                    <driver name="h2" module="com.h2database.h2">
                        <xa-datasource-class>org.h2.jdbcx.JdbcDataSource</xa-datasource-class>
                    </driver>
                </drivers>
            </datasources>
        </subsystem>

* Infinispan subsystem definition:

        <subsystem xmlns="urn:infinispan:server:core:6.0" default-cache-container="local">
            <cache-container name="local" default-cache="default">
                <local-cache name="default" start="EAGER">
                    <locking isolation="NONE" acquire-timeout="30000" concurrency-level="1000" striping="false"/>
                    <transaction mode="NONE"/>
                </local-cache>
                <local-cache name="memcachedCache" start="EAGER">
                    <locking isolation="NONE" acquire-timeout="30000" concurrency-level="1000" striping="false"/>
                    <transaction mode="NONE"/>
                </local-cache>
                <local-cache name="namedCache" start="EAGER"/>
                
                <!-- ADD a local cache called 'addressbook' -->
               
                <local-cache name="addressbook" start="EAGER" batching="false">
                    
                    <!-- Define the locking isolation of this cache -->
                    <locking
                        isolation="READ_COMMITTED"
                        acquire-timeout="20000"
                        concurrency-level="500"
                        striping="false" />
                        
                    <!-- Disable transactions for this cache -->
                    <transaction mode="NONE" />

                    <!-- Enable indexing using the RAM Lucene directory provider -->
                    <indexing index="ALL">
                        <property name="default.directory_provider">ram</property>
                        <property name="lucene_version">LUCENE_36</property>
                    </indexing>
                    
                    <!-- Define the JdbcBinaryCacheStores to point to the ExampleDS previously defined -->
                    <string-keyed-jdbc-store datasource="java:jboss/datasources/ExampleDS" passivation="false" preload="false" purge="false">

                        <!-- specifies information about database table/column names and data types -->
                        <string-keyed-table prefix="JDG">
                            <id-column name="id" type="VARCHAR"/>
                            <data-column name="datum" type="BINARY"/>
                            <timestamp-column name="version" type="BIGINT"/>
                        </string-keyed-table>
                    </string-keyed-jdbc-store>
                </local-cache>
                <!-- End of local cache called 'addressbook' definition -->

            </cache-container>
        </subsystem>

Start JDG
---------

1. Open a command line and navigate to the root of the JDG directory.
2. The following shows the command line to start the server with the web profile:

        For Linux:   $JDG_HOME/bin/standalone.sh
        For Windows: %JDG_HOME%\bin\standalone.bat


Build and Run the Quickstart
----------------------------

_NOTE: The following build command assumes you have configured your Maven user settings. If you have not, you must include Maven setting arguments on the command line. See [Build and Deploy the Quickstarts](../../README.md#build-and-deploy-the-quickstarts) for complete instructions and additional options._

1. Make sure you have started the JDG as described above.
2. Open a command line and navigate to the root directory of this quickstart.
3. Type this command to build and deploy the archive:

        mvn clean package 
                
4. This will create a file at `target/jboss-remote-query-quickstart.jar`

5. Run the example application in its directory:

        mvn exec:java
 

Using the application
---------------------
Basic usage scenarios can look like this (keyboard shortcuts will be shown to you upon start):

    Available actions:
    0. Display available actions
    1. Add person
    2. Remove person
    3. Add phone
    4. Remove phone
    5. Print all
    6. Query persons by name
    7. Query persons by phone
    8. Quit

        
Type `8` to exit the application.

Run application with different classpath
----------------------------------------
It's possible to run this quickstart with different classpath (other than default created by mvn exec:java),
for instance with ${infinispan-server}/client/hotrod/java classpath.
To do this, compile quickstart with:

        mvn clean package -Pcustom-classpath -Dclasspath=/custom/classpath

This will create a file at `target/jboss-remote-query-quickstart.jar`.
Then you can run it with:

        java -jar target/jboss-remote-query-quickstart.jar

Debug the Application
------------------------------------

If you want to debug the source code or look at the Javadocs of any library in the project, run either of the following commands to pull them into your local repository. The IDE should then detect them.

    mvn dependency:sources
    mvn dependency:resolve -Dclassifier=javadoc

