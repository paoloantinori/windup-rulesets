##### how to run tests:
```
mvn clean test -DrunTestsMatching='jbossfuse|camel'
```

##### KCS reference to deprecated components in JBoss Fuse 7
https://access.redhat.com/articles/3009361


##### Current status

|Rule              |Generic |Java   |XML    | Rule_ids
|-------           |--------|-------|-------|----------
|camel fabric      |   N/A  |  x    |  x    |camel-001, camel-002, camel-003, camel-004
|curator           |   N/A  |  x    |  x    |fuse-003, fuse-004
|fabric8.api       |   N/A  |  x    |  x    |fuse-001, fuse-002
|ports             |        |       |       |
|threads           |        |       |       |
|Jetty             |        |       |       |
|ConfigMap         |        |       |       |
|Logging           |        |       |       |
|password          |        |       |       |
|servicewrapper    |        |       |       |
|zookeeper         |        |       |       |
|emb. broker       |        |       |       |
|master/slave      |        |       |       |
|gateway           |        |       |       |
|jolokia           |        |       |       |
|Geronimo          |        |       |       |
|IntegrationPack   |        |       |       |
|karaf cli commands|        |       |       |
|Switchyard        |        |       |       |
|Apache OpenJPA    |        |       |       |
|BPEL              |        |       |       |
|Camel JBPM        |        |       |       |
|LevelDB           |        |       |       |
|Netty             |        |       |       |
|DTGov             |        |       |       |
|RTGov             |        |       |       |
|Google App Engine |        |       |       |
|Smooks            |        |       |       |
|Spring-DM         |        |       |       |
|S-RAMP            |        |       |       |
|Tanuki Wrapper    |        |       |       |
|patch             |        |       |       |
|Camel-Infinispan  |        |       |       |
|fab               |        |       |       |
|jbi               |        |       |       |
|Tanuki Wrapper    |        |       |       |



# Various notes and comments
- ports
- cluster usage
- threads
- jetty
- ConfigMap (can't use all chars)
- Logging (has to go to stdout, no multiple ones)
- password --> secrets
- servicewrapper -> k8s service ready/heath check
- zookeeper
- activemq
- standalone master/slave
- gateway
- jolokia --> expose service



        <rule id="environment-dependent-calls-03500">
            <when>
                <file filename="jndi.properties" as="default"/>
            </when>
            <perform>
                <iteration>
                    <classification issue-display-mode="all" title="JNDI properties file" category-id="mandatory" effort="1">
                        <description>
                          <![CDATA[
                          The JNDI automatically reads the application resource files from all components in the applications' classpaths.
                          The JNDI then makes the properties from these files available to the service providers.

                          Please ensure the property values listed in this file are available to JBoss.
                          ]]>
                        </description>
                        <tag>webservice</tag>
                    </classification>
                </iteration>
            </perform>
        </rule>

        <when>
            <filecontent filename="{*}" pattern="oracle.net.wallet_location" />
        </when>

        <rule id="weblogic-eap7-017000">
            <when>
                <!-- https://issues.jboss.org/browse/WINDUP-615 -->
                <filecontent filename="{*}.{ext}" pattern="{t3url}" />
            </when>
            <perform>
                <hint title="WebLogic T3 JNDI binding" effort="3" category-id="mandatory">
                    <message>
                    <![CDATA[
                    Weblogic’s implementation of the RMI specification uses a proprietary protocol known as T3. T3S is the version of the protocol over SSL.
                     `t3://` and `t3s://` URLs are used to configure a JNDI InitialContext within WebLogic.

                    The equivalent functionality needs to be configured in JBoss EAP 7.
                    This could be done either by using standard Java EE JNDI names or by using a WebLogic proprietary library if the connectivity to WebLogic server is still required.
                    ]]>
                    </message>
                    <link href="https://docs.oracle.com/cd/E24329_01/web.1211/e24389/rmi_t3.htm#WLRMI143" title="Oracle WebLogic RMI with T3" />
                    <link href="https://access.redhat.com/solutions/1230143" title="Invoking EJBs deployed on WebLogic from EAP6" />
                    <tag>configuration</tag>
                    <tag>weblogic</tag>
                </hint>
            </perform>
            <where param="ext">
                <matches pattern="(java|properties|xml)" />
            </where>
            <where param="t3url">
                <matches pattern="t3s?://" />
            </where>
        </rule>


        <rule id="websphere-mq-eap7-01000">
            <when>
                <!-- https://issues.jboss.org/browse/WINDUP-615 -->
                <filecontent filename="{*}.{ext}" pattern="com.ibm.mq.jms.context.WMQInitialContextFactory" />
            </when>
            <perform>
                <classification issue-display-mode="detail-only" title="IBM MQ Configuration">
                    <description>The WebSphere MQ client API is used to communicate with the MQ server from client-side applications.
                        For JBoss EAP 7, this needs to be replaced with standard Java EE 7 JMS API, or with ActiveMQ Artemis client API.
                    </description>
                </classification>
                <hint title="IBM JMS implementation of WMQInitialContextFactory" category-id="mandatory" effort="3">
                    <message>`WMQInitialContextFactory` is an implementation of `InitialContextFactory`
                        used to get object instances from JNDI. The
                        equivalent functionality needs to be configured on JBoss EAP 7 using ActiveMQ Artemis.
                        `InitialContextFactory` is provided
                        by EAP and you only need to instantiate `InitialContext ctx = new InitialContext();`.
                    </message>
                    <link title="The Embedded ActiveMQ Artemis Messaging Broker"
                          href="https://access.redhat.com/documentation/en-us/red_hat_jboss_enterprise_application_platform/7.0/html-single/configuring_messaging/#the_integrated_activemq_artemis_messaging_broker" />
                        <link title="Java EE 7 JMS Tutorial" href="https://docs.oracle.com/javaee/7/tutorial/jms-concepts003.htm#BNCEH" />
                    <tag>jms</tag>
                    <tag>messaging</tag>
                    <tag>configuration</tag>
                    <tag>websphere</tag>
                </hint>
            </perform>
            <where param="ext">
                <matches pattern="(java|properties|xml)" />
            </where>
        </rule>

        <rule id="websphere-jms-eap7-02500">
            <when>
                <javaclass references="com.ibm.mqe.jms.{type}Queue" />
            </when>
            <perform>
                <hint title="WebSphere implementation MQe{type}Queue of JMS Queue" category-id="mandatory" effort="1">
                    <message>
                        `MQe{type}Queue` is a WebSphere implementation of a JMS `Queue` and should be migrated to
                        the Java EE 6 JMS standard interface `javax.jms.Queue`.
                    </message>
                    <link title="Java EE 7 JMS Tutorial" href="https://docs.oracle.com/javaee/7/tutorial/jms-concepts003.htm#BNCEH" />
                    <link title="Java EE 7 JMS Tutorial" href="https://docs.oracle.com/javaee/7/tutorial/jms-concepts003.htm#BNCEH" />
                    <tag>jms</tag>
                    <tag>websphere</tag>
                </hint>
            </perform>
        </rule>

        <rule id="eap6-08000">
            <when>
                <filecontent pattern="remote://{node}:{port}" filename="{*}.{extension}" />
            </when>
            <perform>
                <hint title="Remote JNDI Provider URL has changed in EAP 7" effort="1" category-id="mandatory">
                    <message>
                        Default Remote JNDI Provider URL has changed in EAP 7. External applications using JNDI to lookup remote resources, for instance an EJB or a JMS Queue,
                        may need to change the value for the JNDI InitialContext environment's property named `java.naming.provider.url`.
                        The default URL scheme is now **http-remoting** instead of **remote**, and the default URL port is now **8080** instead of **4447**.

                        As an example, consider the application server host is localhost, then clients previously accessing EAP 6 would use

                        ```
                        java.naming.factory.initial=org.jboss.naming.remote.client.InitialContextFactory
                        java.naming.provider.url=remote://localhost:4447
                        ```

                        while clients now accessing EAP 7 should use instead

                        ```
                        java.naming.factory.initial=org.jboss.naming.remote.client.InitialContextFactory
                        java.naming.provider.url=http-remoting://localhost:8080
                        ```

                    </message>
                    <link href="https://access.redhat.com/documentation/en/red-hat-jboss-enterprise-application-platform/version-7.0/migration-guide/#migrate_default_remote_url_connector_and_port_changes" title="Remote JNDI URL in EAP 7" />
                    <tag>jndi</tag>
                    <tag>configuration</tag>
                    <tag>ejb</tag>
                </hint>
            </perform>
            <where param="node">
                <matches pattern=".*"/>
            </where>
            <where param="port">
               <matches pattern="\d*"/>
            </where>
            <where param="extension">
                <matches pattern="(java|properties|xml)"/>
            </where>
        </rule>