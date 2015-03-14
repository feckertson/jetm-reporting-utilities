# Introduction #

The JETM reporting utilities provide an abstract definition of a JETM [MeasurementRenderer](http://jetm.void.fm/api/etm/core/renderer/MeasurementRenderer.html), called a [BindingMeasurementRenderer](http://jetm-reporting-utilities.googlecode.com/svn/tags/jetm-reporting-utilities-1.0/src/main/java/com/google/code/jetm/reporting/BindingMeasurementRenderer.java). The concept is this: you provide an implementation of an [AggregateBinder](http://jetm-reporting-utilities.googlecode.com/svn/tags/jetm-reporting-utilities-1.0/src/main/java/com/google/code/jetm/reporting/AggregateBinder.java), which does the actual work of writing out JETM measurement data, and a java.io.Writer. The BindingMeasurementRenderer acts as a glue between the abstraction (the AggregateBinder) and the existing JETM framework.

You can use these utilities with the [jetm-maven-plugin](http://code.google.com/p/jetm-maven-plugin/) to generate reports from this data.

That's a lot of words, so let's see some examples.

# XmlAggregateBinder Example #

This project provides an XML-driven binder which writes out its timing report data as XML data. The following test shows how to write out data using this binder:

```
package foo;

import etm.core.configuration.BasicEtmConfigurator;
import etm.core.configuration.EtmManager;
import etm.core.monitor.EtmMonitor;

import com.google.code.jetm.reporting.BindingMeasurementRenderer;
import com.google.code.jetm.reporting.xml.XmlAggregateBinder;

public class XmlAggregateBinderDemoTest {
    private static EtmMonitor monitor;

    /**
     * Configure JETM
     */
    @BeforeClass
    public static void setUpBeforeClass() {
        BasicEtmConfigurator.configure();

        monitor = EtmManager.getEtmMonitor();
        monitor.start();
    }

    /**
     * Write out the results of all of the test runs. This writes out 
     * the collected point data to an XML file located in target/jetm
     * beneath the working directory.
     * 
     * @throws Exception
     *             If any errors occur during the write-out.
     */
    @AfterClass
    public static void tearDownAfterClass() throws Exception {
        if (monitor != null) {
            monitor.stop();

            final File timingDirectory = new File("target/jetm");
            FileUtils.forceMkdir(timingDirectory);

            final File timingFile = new File(timingDirectory, XmlAggregateBinderDemoTest.class.getSimpleName() + ".xml");
            final FileWriter writer = new FileWriter(timingFile);
            try {
                monitor.render(new BindingMeasurementRenderer(new XmlAggregateBinder(), writer));
            } finally {
                writer.close();
            }
        }
    }

    @Test
    public void doStuff() {
       // execute some code here that starts and stops JETM collection points
    }
}
```