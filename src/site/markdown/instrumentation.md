<!---
Licensed to the Apache Software Foundation (ASF) under one
or more contributor license agreements.  See the NOTICE file
distributed with this work for additional information
regarding copyright ownership.  The ASF licenses this file
to you under the Apache License, Version 2.0 (the
"License"); you may not use this file except in compliance
with the License.  You may obtain a copy of the License at

  http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing,
software distributed under the License is distributed on an
"AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
KIND, either express or implied.  See the License for the
specific language governing permissions and limitations
under the License.
-->
# Manually (clients)

To handle manually the interception you need to import commons-monitoring-aop.
Then you can rely on `org.apache.commons.monitoring.aop.MonitoringProxyFactory`.

`org.apache.commons.proxy.ProxyFactory` key defines the proxy factory to use to create proxies For instance
to use javassist you set it to `org.apache.commons.proxy.factory.javassist.JavassistProxyFactory`
(and you'll include javassist in your application).

Then the API is quite simple:

    final MyClient client = MonitoringProxyFactory.monitor(MyClient.class, getMyClientInstance());

# CDI

You just need to decorate your CDI bean/method with the interceptor binding `org.apache.commons.monitoring.cdi.Monitored`.

For instance:


    @Monitored
    @ApplicationScoped
    public class MyMonitoredBean {
        public void myMethod() {
            // ...
        }
    }

Note: in some (old) CDI implementation you'll need to activate the monitoring interceptor: `org.apache.commons.monitoring.cdi.CommonsMonitoringInterceptor`.

Note: we are working to make it configurable.

# Spring

Using `org.apache.commons.monitoring.spring.BeanNameMonitoringAutoProxyCreator` you can automatically
add monitoring to selected beans.

    <bean class="org.apache.commons.monitoring.spring.BeanNameMonitoringAutoProxyCreator">
      <property name="beanNames">
        <list>
          <value>*Service</value>
        </list>
      </property>
    </bean>

An alternative is to use `org.apache.commons.monitoring.spring.PointcutMonitoringAutoProxyCreator` which uses
a `org.springframework.aop.Pointcut` to select beans to monitor.

# AspectJ

To use AspectJ weaver (it works with build time enhancement too but it is often less relevant) just configure a custom
concrete aspect defining the pointcut to monitor:

    <aspectj>
      <aspects>
        <concrete-aspect name="org.apache.commons.aspectj.MyMonitoringAspectJ"
                         extends="org.apache.commons.monitoring.aspectj.CommonsMonitoringAspect">
          <pointcut name="pointcut" expression="execution(* org.apache.commons.monitoring.aspectj.AspectJMonitoringTest$MonitorMe.*(..))"/>
        </concrete-aspect>
      </aspects>

      <weaver>
        <include within="org.apache.commons.monitoring.aspectj.AspectJMonitoringTest$MonitorMe"/>
      </weaver>
    </aspectj>

See [AspectJ documentation](http://eclipse.org/aspectj/doc/next/progguide/language-joinPoints.html) for more information.