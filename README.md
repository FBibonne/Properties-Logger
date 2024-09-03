# Properties Logger

_Properties Logger_ is a module for Spring Boot 3+ apps which early logs properties
detected by Spring Boot and their values resolved by Spring Boot.

## Requirements

The _properties Logger_ module is designed to work with :
- Spring Boot 3.3+ (an so java 17)
- Servlet web application
- Reactive web application (not tested)
- Batch application application (not tested)
- CommandLineRunner application (not tested)
- Spring boot version < 3.3 (not tested)

## Usage

Usage is simple : just add this dependency inside your pom.xml :
```xml
<dependency>
    <groupId>fr.insee</groupId>
    <artifactId>boot-properties-logger-starter</artifactId>
    <version>1.0.0</version>
</dependency>
```

NB : 

1. the lib is not deployed on maven central for the moment so you must install
it locally before using it :
```shell
$ git clone https://github.com/FBibonne/Properties-Logger.git --depth=1 --branch=master
$ mvn install -f Properties-Logger/pom.xml
```

2. The module module _Properties Logger_ logs its message with properties and their values
at the info level : so its **log level must be at least INFO**. DEBUG (or TRACE) give (much) more
informations.

## Result

You should see this kind of output in your log (in the console by default in a Spring Boot app) :
```text
2024-06-25T12:01:00.858+02:00  INFO 42091 --- [           main] fr.insee.boot.PropertiesLogger           : 
================================================================================
                        Values of properties from sources :
- Config resource 'class path resource [application.properties]' via location 'optional:classpath:/'
                                     ====
springdoc.show-actuator = true
springdoc.swagger-ui.path = /
springdoc.pathsToMatch = /**
springdoc.swagger-ui.oauth.clientId = 
server.forward-headers-strategy = framework
spring.security.oauth2.resourceserver.jwt.jwk-set-uri = 
spring.datasource.username = user
spring.datasource.password = ******
springdoc.swagger-ui.syntax-highlight.activated = false
management.endpoint.health.show-details = always
logging.level.root = INFO
================================================================================

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/

 :: Spring Boot ::                (v3.3.1)
```

The first part of the log message generated by the module (before the separator `====` )
lists the sources from wich properties keys are collected : only the property sources
which are enumerable and [which are not excluded](#excluded-properties-sources) are listed there

The second part of the log message (after the separator `====` ) lists collected properties
key which starts with [one element of the prefix list](#prefix-list-for-displayed-properties) 
and its value resolved by Spring Boot. The displayed value **is not necessarily** the one
which is defined in the property source where Spring Boot encoutered the key, neither one
of the values defined in other displayed property source but the real one that Spring Boot
would provide to your application (for example in a `@Value` annotation) following [its 
property resolver algorithm](https://docs.spring.io/spring-boot/reference/features/external-config.html)

For example, given the property key `spring.datasource.username`. If it is defined in the 
application.properties file with `user_dev` value :
```properties
spring.datasource.username=user_dev
```

But the environement variable `SPRING_DATASOURCE_USERNAME` is also defined with the value
`user_prod`.

The _Properties Logger_ module will display the property `spring.datasource.username` (because
by default the keys listed in the application.properties file ar listed) but with the value of
`user_prod` as resolved by Spring Boot due to the environment variable. The environment variable
is not listed because by default the property keys in the system environment property source (`systemEnvironment`)
are not displayed.

Values of [properties with secrets](#properties-with-hidden-values) are hidden when they are printed :
`******` is displayed instead. For example `spring.datasource.password = ******`. To be hidden, the 
property key must contain one of the words from the list [properties with secrets](#properties-with-hidden-values).

## Configuration

We describe here configurations which can be applied to the module _Properties Logger_ via 
properties prefixed by `properties.logger`. They are three of them:
- [`properties.logger.sources-ignored`](#excluded-properties-sources)
- [`properties.logger.prefix-for-properties`](#prefix-list-for-displayed-properties)
- [`properties.logger.with-hidden-values`](#properties-with-hidden-values)

### Excluded properties sources

| Related Property                    | Default value                        |
|-------------------------------------|:-------------------------------------|
| `properties.logger.sources-ignored` | systemProperties, systemEnvironment  |

At starting, Spring Boot can process many property sources and not all of them  provide values for
the properties used in your application. Particularly the system properties (java properties which 
can be read with `System#getProperty`) and the OS environment variables (can be read with `System#getenv`)
contain many key-value pairs which you do not directly use in your application : displaying 
them make the log too much verbose. So you can exclude the properties key exclusively defined by these property
sources listing them in the  `properties.logger.sources-ignored`.

By default the system properties (`systemProperties`) and the environment variables(`systemEnvironment`)
are excluded. You can exclude more properties source by adding their names to the list. 

> For example, to not not take into account property keys from :
> 1. the file application.properties
> 2. a file with a relative path ` ../secrets/secret.properties`
> 3. the command line arguments
> 4. the `properties` attribute of `@SpringBootTest` and others @*Test annotations,
> you should set these values :

```properties
properties.logger.sources-ignored= systemProperties, systemEnvironment,\
[application.properties],\
../secrets/secret.properties,\
commandLineArgs,\
Inlined\ Test\ Properties
```

> At first line, set default exclusions (RECOMMENDED) then add others exclusions :
>
> 2. at second line, for the the application.properties file,
> 3. at third line for ../secrets/secret.properties file
> 4. at fourth line for command ligne arguments
> 5. at fifth line for then properties attributes for tests

### Prefix list for displayed properties

**Feature controller by `properties.logger.prefix-for-properties`**

### Properties with hidden values

**Feature controller by `properties.logger.with-hidden-values`**
