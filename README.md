# VWO JAVA SDK

[![Build Status](http://img.shields.io/travis/wingify/vwo-java-sdk/master.svg?style=flat)](http://travis-ci.org/wingify/vwo-java-sdk)
[![Coverage Status](https://coveralls.io/repos/github/wingify/vwo-java-sdk/badge.svg?branch=master)](https://coveralls.io/github/wingify/vwo-java-sdk?branch=master)

This open source library allows you to A/B Test your Website at server-side.

## Requirements

The Java SDK supports:

* Open JDK 8, 9, 11, 12
* Oracle JDK 8, 9, 11, 12

## SDK INSTALLATION

Install dependencies using 'mvn install'

Add below Maven dependency in your project.


```java
<dependency>
    <groupId>com.vwo.sdk</groupId>
    <artifactId>vwo-java-sdk</artifactId>
    <version>1.0.0</version>
</dependency>
```

## Documentation

Refer [Official VWO Documentation](https://developers.vwo.com/reference#server-side-introduction)

## Basic Usage

**GET SETTING FILE**

Each VWO SDK client corresponds to the settingsFIle representing the current state of the campaign settings, that is, a list of server-side running campaign settings.
Setting File is a pre-requisite for initiating the VWO CLIENT INSTANCE.

```java
String settingsFile = VWO.getSettingsFile(accountId, sdkKey));
```

**INSTANTIATION**


SDK provides a method to instantiate a VWO client as an instance. The method accepts an object to configure the VWO client.
The mandatory parameter for instantiating the SDK is settingsFile.

```java
import com.vwo.VWO;

VWO vwoInstance = VWO.createInstance(settingsFile).build();
```

The VWO client class needs to be instantiated as an instance that exposes various API methods like activate, getVariation and track.

**ASYNC EVENT DISPATCHER**

```java
import com.vwo.event.EventDispatcher;
import com.vwo.config.FileSettingUtils;


public class Example {

    private final VWO vwo;

    public Example(VWO vwo) {
        this.vwo = vwo;
    }

    public static void main(String[] args) {

        String settingsFile = VWO.getSettingsFile(accountId, sdkKey);

        EventDispatcher eventDispatcher = EventDispatcher.builder().build();

        VWO vwo_instance = VWO.createInstance(settingsFile).withEventHandler(eventDispatcher).build();
    }
}
```

**USER PROFILE SERVICE**

```java
String settingsFile = VWO.getSettingsFile(accountId, sdkKey);

UserProfileService userProfileService = new UserProfileService() {
    @Override
    public Map<String, Object> lookup(String s, String s1) throws Exception {
    // hardcode values in map
    // one can get values from  db and populate map

        String campaignId = "FIRST";
        String variationId = "Control";

        Map<String,Object> campaignKeyMap = new HashMap<>();
        Map<String, String> variationKeyMap = new HashMap<>();
        variationKeyMap.put(UserProfileService.variationKey, variationId);
        campaignKeyMap.put(campaignId,variationKeyMap);

        //set
        Map<String, Object> campaignStaticBucketMap = new HashMap<>();
        campaignStaticBucketMap.put(UserProfileService.userId, "Priya");
        campaignStaticBucketMap.put(UserProfileService.campaignKey, campaignKeyMap);

        return campaignStaticBucketMap;
    }

    @Override
    public void save(Map<String, Object> map) throws Exception {
    // save in db or some data store

    }
};

VWO vwo = VWO.createInstance(settings).withUserProfileService(userProfileService).build();
```

**LOGGER**

JAVA SDK utilizes a logging facade, SL4J (https://www.slf4j.org/) as the logging api layer. If no binding is found on the class path,
then you will see the following logs

```
SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
You can get the SDK to log by providing a concrete implementation for SLF4J.
```

What it means is that at runtime, the logging `implementation` (or the logger binding) is missing , so slf4jsimply use a "NOP" implmentation, which does nothing.
If you need to output JAVA SDK logs, there are different approaches for the same.

SIMPLE IMPLEMENTATION
If there are no implementation in your project , you may provide a simple implementation that does not require any configuration at all.
Add following code to you pom.xml,

```java
 <dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-simple</artifactId>
    <version>1.6.4</version>
</dependency>
```

Now you see logging output on STDOUT with INFO level. This simple logger will default show any INFO level message or higher.
In order to see DEBUG messages, you would need to pass -Dorg.slf4j.simpleLogger.defaultLogLevel=debug or simplelogger.properties file on the classpath
See http://www.slf4j.org/api/org/slf4j/impl/SimpleLogger.html for details

CONCRETE IMPLEMENTATION

sl4j supports various logging framework. Refer here ->https://www.slf4j.org/manual.html

We have provided our example with Logback
If you have logback in your class path, to get console logs add following Appender and Logger in logback for formatted logs.

```java
<appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
        <Pattern>
            %cyan(VWO-SDK) [%date] %highlight([%level]) %cyan([%logger{10} %file:%line]) %msg%n
        </Pattern>
    </encoder>

    <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
        <level>DEBUG</level>
    </filter>
</appender>

<Logger name="com.vwo.sdk" additivity="false">
    <appender-ref ref="STDOUT"/>
</Logger>
```

```java
public static VWOLogger getCustomLogger() {
    return new VWOLogger(VWO.Enums.LOGGER_LEVEL.DEBUG.value()) {

        @Override
        public void trace(String message, Object... params) {
            LOGGER.trace(message, params);
        }

        @Override
        public void debug(String message, Object... params) {
            LOGGER.debug(message, params);
        }

        @Override
        public void info(String message, Object... params) {
            LOGGER.info(message, params);
        }

        @Override
        public void warn(String message, Object... params) {
            LOGGER.warn(message, params);
        }

        @Override
        public void error(String message, Object... params) {
            LOGGER.error(message, params);
        }
    };
}
```

For more appenders, refer [this](https://logback.qos.ch/manual/appenders.html).

## Authors

* Core Contributir & Maintainer - [pntgupta](https://github.com/pntgupta)
* Main Contributor - [sakshimahendruvk](https://github.com/sakshimahendruvk)
* Repo health maintainer - [softvar](https://github.com/softvar)

## Contributing

Please go through our [contributing guidelines](https://github.com/wingify/vwo-java-sdk/CONTRIBUTING.md)

## Code of Conduct

[Code of Conduct](https://github.com/wingify/vwo-java-sdk/blob/master/CODE_OF_CONDUCT.md)

## License

```text
    MIT License

    Copyright (c) 2019 Wingify Software Pvt. Ltd.

    Permission is hereby granted, free of charge, to any person obtaining a copy
    of this software and associated documentation files (the "Software"), to deal
    in the Software without restriction, including without limitation the rights
    to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
    copies of the Software, and to permit persons to whom the Software is
    furnished to do so, subject to the following conditions:

    The above copyright notice and this permission notice shall be included in all
    copies or substantial portions of the Software.

    THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
    IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
    FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
    AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
    LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
    OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
    SOFTWARE.
```
