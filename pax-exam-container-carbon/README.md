# In-Container OSGi Testing

> This section explains the Pax Exam container in Carbon, which is used for OSGi testing. For the list of capabilities available in Carbon Kernel 5.1.0, see the [root README.md file](https://github.com/nilminiwso2/carbon-kernel-1/blob/master/README.md).

From Carbon Kernel 5.0.0 onwards, the [Pax Exam framework](https://ops4j1.jira.com/wiki/display/PAXEXAM4/Getting+Started+with+OSGi+Tests) provides the infrastructure for composing unit test cases. See the following topics for details:

* **[How Pax Exam works](#how-pax-exam-works)**
* **[Writing unit tests for a Carbon component](#writing-unit-tests-for-a-carbon-component)**

## How Pax Exam works

Pax Exam starts up the OSGi framework (which is equinox for the Carbon kernel) with the minimum level of bundles that are necessary for the pax-exam operations. Then the Pax Exam user can provision the bundles to the OSGi container. When the Pax Exam boots up, these provisioned bundles will be installed in the container. The list of bundles that are provisioned by default in a Carbon server are given [here](https://github.com/wso2/carbon-kernel/blob/v5.1.0/tests/osgi-test-util/src/main/java/org/wso2/carbon/osgi/test/util/OSGiTestConfigurationUtils.java). However, when you develop a Carbon component, you will be able to change the default setting by provisioning all the required bundles for your component.

## Writing unit tests for a Carbon component
Follow the steps given below to incorporate unit test cases for your Carbon component.

### Step 1: Changing the default Pax Exam configurations

You can change the default Pax Exam configurations by following the steps given below.

1. Change the `pax.exam.system` property in the `pom.xml` file of the OSGi test component from 'test' to 'default' as shown below. 
 > As mentioned in the [Pax Exam Configuration](https://ops4j1.jira.com/wiki/display/PAXEXAM4/Configuration) documentation, Pax Exam starts the OSGi container in 'test' mode with a standard set of options that are compatible with Pax Exam 2.3.0. This setting will provision only the default set of bundles listed [here](https://github.com/wso2/carbon-kernel/blob/v5.1.0/tests/osgi-test-util/src/main/java/org/wso2/carbon/osgi/test/util/OSGiTestConfigurationUtils.java) for your component. Therefore, using the `default` mode allows you to add other dependencies for your component, in addition to the default bundles.

        <build>
        …….
           <plugins>
           …….
              <plugin>
                 <groupId>org.apache.maven.plugins</groupId>
                 <artifactId>maven-surefire-plugin</artifactId>
                 <configuration>
                    <systemPropertyVariables>
                       …….
                       <pax.exam.system>default</pax.exam.system>
                       …….
                    </systemPropertyVariables>
                       …….
                    <systemProperties>
                 …….
                 </configuration>
              …….
              </plugin>
           …….
           </plugins>
        …….
        </build>

2. Update the `pom.xml` of your Carbon component with the following dependencies:

 * Dependency for OSGi Test Utils:
 
           <dependency>
               <groupId>org.wso2.carbon</groupId>
               <artifactId>osgi-test-util</artifactId>
               <version>5.1.0</version>
            </dependency>
 
 * Other required dependencies for your component.

3. Optionally, you can change the default log level in Pax Exam (which is 'debug') by adding the `“org.ops4j.pax.logging.DefaultServiceLog.level”` system property to the pom.xml file of the OSGi test component as shown below.

        <build>
        …….
         <plugins>
         …….
         <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <configuration>
                 <systemPropertyVariables>
                 …….
                 <org.ops4j.pax.logging.DefaultServiceLog.level>ERROR</org.ops4j.pax.logging.DefaultServiceLog.level>
                 …….
                 </systemPropertyVariables>
                   …….
                 <systemProperties>
                 …….
                 </configuration>
         …….
          </plugin>
         …….
         </plugins>
        …….
        </build>

### Step 2: Writing test cases for the Carbon component

Follow the instructions given below when you write test cases for your Carbon component.

1. Start writing your test case as shown in the sample test case given below. From this sample, you can see that the test case is similar to a normal test case while it contains some annotations related to Pax Exam.

 Pax Exam will boot the OSGi framework with the necessary bundles for your test environment. You have to use the `@RunWith(PaxExam.class)` annotation to hook Pax Exam into testing. Pax Exam will then set up a test container with an OSGi framework and your bundles to run the tests.

 > Be sure to inject the [`carbonServerInfo`](https://github.com/wso2/carbon-kernel/blob/v5.1.0-rc1/core/src/main/java/org/wso2/carbon/kernel/utils/CarbonServerInfo.java) service when you write your test case. Without this service, the framework will not fully start. The sample test case shown below illustrates how this service is injected. 

 *Sample test case:* 

        package org.wso2.carbon.osgi;

        import org.ops4j.pax.exam.spi.reactors.ExamReactorStrategy;
        import org.ops4j.pax.exam.spi.reactors.PerClass;
        import org.ops4j.pax.exam.testng.listener.PaxExam;
        import org.osgi.framework.Bundle;
        import org.osgi.framework.BundleContext;
        import org.testng.Assert;
        import org.testng.annotations.Listeners;
        import org.testng.annotations.Test;

        import javax.inject.Inject;

        @Listeners(PaxExam.class)
        @ExamReactorStrategy(PerClass.class)
        public class BaseOSGiTest {

        @Inject
        private BundleContext bundleContext;

         @Inject
        private CarbonServerInfo carbonServerInfo;

	        @Test
        public void testBundleContextStatus() {
          Assert.assertNotNull(bundleContext, "Bundle Context is null");
         }

        @Test
        public void testCarbonCoreBundleStatus() {

          Bundle coreBundle = null;
         for (Bundle bundle : bundleContext.getBundles()) {
                if (bundle.getSymbolicName().equals("org.wso2.carbon.core")) {
                 coreBundle = bundle;
                 break;
                }
          }
         Assert.assertNotNull(coreBundle, "Carbon Core bundle not found");
         Assert.assertEquals(coreBundle.getState(), Bundle.ACTIVE, "Carbon Core
                                                             Bundle  is not activated") ;
        }
        }

 Using Dependency Injection, your test method can access the `BundleContext` of the probe bundle or any service obtained from the OSGi service registry.
 
		@Inject
		private BundleContext bundleContext;

 Alternatively, you can directly access the OSGi services using the following:
 
		 @Inject
		private SomeSampleOsgiService service; 

2. If you need any bundles other than those defined here, you can add them to the test case class. The following are the list of APIs provided by the OSGi Test Utils that you can use:

 * `getConfiguration() : (returns List<Option>)`
	
    This method is used to get the Pax Exam configuration with the default (minimum) Pax Exam options required for booting up the Carbon Kernel.
 
 * `getConfiguration(List<Option> customOptions, CarbonSysPropConfiguration sysPropConfiguration) : (returns List<Option>)`
	
    This method is used to pass custom options (`customOptions`) and `systemproperty` configurations (`sysPropConfiguration`). Here, the custom options are merged with the default set of options required to boot up Carbon Kernel. Using `sysPropConfiguration`, you can set the server key,  server name,  server version and carbon home.  
 
 * `getBaseOptions(String carbonHome, String serverKey, String serverName, String serverVersion) : (returns List<Option>)`
	
    This method is used to set a systemproperty such as 'carbon home', 'server key', 'server name' and 'server version' while using the default (minimum) Pax Exam options required to start the server.
 
 All of the above will return a list of options (`org.ops4j.pax.exam.Option`) that you have returned as an array. Note that you need to use the method inside `@Configuration` method (`org.ops4j.pax.exam.Configuration`).
 
 *Example 1:*   
 
		 import org.ops4j.pax.exam.Configuration;
		import org.ops4j.pax.exam.Option;
		import org.wso2.carbon.osgi.test.util.OSGiTestConfigurationUtils;
		…….
		@Configuration
		public Option[] createConfiguration() {
  		 List<Option> optionList = OSGiTestConfigurationUtils.getConfiguration();
   		copyCarbonYAML();	//Custom method to change carbon.yml file
   		return optionList.toArray(new Option[optionList.size()]);
		
 *Example 2:*   
 
		 import org.ops4j.pax.exam.Configuration;
		import org.ops4j.pax.exam.Option;
		import org.wso2.carbon.osgi.test.util.OSGiTestConfigurationUtils;
		…….
		@Configuration
		public Option[] createConfiguration() {
  		 System.setProperty(Constants.TENANT_NAME, TEST_TENANT_NAME);
  		 List<Option> optionList = OSGiTestConfigurationUtils.getConfiguration();
  		 copyConfigFiles();
  		 optionList.add(mavenBundle()
          		 .artifactId("carbon-context-test-artifact")
          		 .groupId("org.wso2.carbon")
         		  .versionAsInProject());
 		  return optionList.toArray(new Option[optionList.size()]);
		}

3. The `carbon.home` (system property) should be set for OSGi tests. This location will vary depending on the repository that is used. Therefore, this property should be defined in the pom.xml of the OSGi test as shown below:

		<build>
		…….
 		  <plugins>
		   …….
		      <plugin>
  		       <groupId>org.apache.maven.plugins</groupId>
  		       <artifactId>maven-surefire-plugin</artifactId>
   		      <configuration>
  		          <systemProperties>
  		             …….
   		            <property>
    		                <name>carbon.home</name>
   		                 <value>${basedir}/target/wso2carbon-kernel-${project.version}</value>
       		        </property>
   		            …….
      		      </systemProperties>
     		    …….
   		      </configuration>
  		    …….
 		     </plugin>
 		  …….
 		  </plugins>
		…….
		</build>
