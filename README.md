[![Maven Central](https://maven-badges.herokuapp.com/maven-central/com.github.sabomichal/immutable-xjc-plugin/badge.svg)](https://maven-badges.herokuapp.com/maven-central/com.github.sabomichal/immutable-xjc-plugin) ![Java CI with Maven](https://github.com/sabomichal/immutable-xjc/workflows/Java%20CI%20with%20Maven/badge.svg)
## immutable-xjc
IMMUTABLE-XJC is a JAXB 2.x XJC plugin for making schema derived classes immutable:

* removes all setter methods
* marks class final (can be disabled with the '-imm-nofinalclasses' option)
* creates a public constructor with all fields as parameters
* creates a protected no-arg constructor
* marks all fields within a class as final
* wraps all collection like parameters with Collections.unmodifiable or Collections.empty if null (unless -imm-leavecollections option is used)
* optionally creates builder pattern utility classes

Note: Derived classes can be further made serializable using these xjc [customizations](http://docs.oracle.com/cd/E17802_01/webservices/webservices/docs/1.6/jaxb/vendorCustomizations.html#serializable).

### JAXB version
Plugin is built against JAXB 4.0.0

### Java version
Target Java versions is 11 

### XJC options provided by the plugin
The plugin provides an '-immutable' option which is enabled by adding its jar file to the XJC classpath. When enabled, additional options can be used to control the behavior of the plugin. See the examples for further information.

#### -immutable
The '-immutable' option enables the plugin making the XJC generated classes immutable.

#### -imm-builder
The '-imm-builder' option can be used to generate builder like pattern utils for each schema derived class.

#### -imm-simplebuildername
The '-imm-simplebuildername' option can be used to generate builders which follow a simpler naming scheme, using Foo.builder() and Foo.Builder instead of Foo.fooBuilder() and Foo.FooBuilder.

#### -imm-inheritbuilder
The '-imm-inheritbuilder' option can be used to generate builder classes that follow the same inheritance hierarchy as their subject classes.

#### -imm-cc
The '-imm-cc' option can only be used together with '-imm-builder' option and it is used to generate builder class copy construstructor, initialising builder with object of given class.

#### -imm-ifnotnull
The '-imm-ifnotnull' option can only be used together with '-imm-builder' option and it is used to add an additional withAIfNotNull(A a) method for all non-primitive fields A in the generated builders.

#### -imm-nopubconstructor
The '-imm-nopubconstructor' option is used to make the constructors of the generated classes non-public.

#### -imm-pubconstructormaxargs
The '-imm-pubconstructormaxargs=n' option is used to generate public constructors with up to n arguments, when -imm-builder is used 

#### -imm-skipcollections
The '-imm-skipcollections' option is used to leave collections mutable

#### -imm-constructordefaults
The '-imm-constructordefaults' option is used to set default values for xs:element's and xs:attribute's in no-argument constructor. Default values must be strings or numbers, otherwise ignored.

#### -imm-optionalgetter
The '-imm-optionalgetter' option is used to wrap the return value of getters for non-required (`@XmlAttribute|Element(required = false)`) values with `java.util.Optional<OriginalRetunType>`.

#### -imm-nofinalclasses
The '-imm-nofinalclasses' option is used to leave all classes non-final.

### Usage
#### JAXB-RI CLI
To use the JAXB-RI XJC command line interface simply add the corresponding java archives to the classpath and execute the XJC main class 'com.sun.tools.xjc.XJCFacade'. The following example demonstrates a working command line for use with JDK 11+ (assuming the needed dependencies are found in the current working directory).
```sh
java.exe -Dcom.sun.tools.xjc.XJCFacade.nohack=true\ 
         -classpath codemodel-2.3.2.jar:\
                    jaxb-api-2.3.2.jar:\
                    jaxb-runtime-2.3.2.jar:\
                    jaxb-xjc-2.3.2.jar:\
                    javax.activation-api-1.2.0.jar:\
                    javax.activation-1.2.0.jar:\
                    rngom-2.3.2.jar:\
                    istack-commons-tools-3.0.7.jar:\
                    istack-commons-runtime-3.0.7.jar:\
                    relaxng-datatype-2.3.2.jar:\
                    txw2-2.3.1.jar:\
                    xsom-2.3.1.jar com.sun.tools.xjc.XJCFacade -immutable <schema files>
```
#### Maven
Maven users simply add the IMMUTABLE-XJC plugin as a dependency to a JAXB plugin of choice. The following example demonstrates the use of the IMMUTABLE-XJC plugin with the mojo *https://github.com/evolvedbinary/jvnet-jaxb-maven-plugin*.
```xml
<plugin>
    <groupId>com.evolvedbinary.maven.jvnet</groupId>
    <artifactId>jaxb30-maven-plugin</artifactId>
    <version>0.15.0</version>
    <dependencies>
        <dependency>
            <groupId>com.github.sabomichal</groupId>
            <artifactId>immutable-xjc-plugin</artifactId>
            <version>1.8.0</version>
        </dependency>
    </dependencies>
    <executions>
        <execution>
            <phase>generate-sources</phase>
            <goals>
                <goal>generate</goal>
            </goals>
            <configuration>
                <specVersion>4.0.0</specVersion>
                <args>
                    <arg>-immutable</arg>
                    <arg>-imm-builder</arg>
                </args>
            </configuration>
        </execution>
    </executions>
</plugin>
```

IMMUTABLE-XJC can be used also in contract-first webservice client scenarios with wsimport. The following example demonstrates the usage of the plugin with *jaxws-maven-plugin* mojo.
```xml
<plugin>
    <groupId>org.jvnet.jax-ws-commons</groupId>
    <artifactId>jaxws-maven-plugin</artifactId>
    <dependencies>
        <dependency>
            <groupId>com.github.sabomichal</groupId>
            <artifactId>immutable-xjc-plugin</artifactId>
            <version>1.7.0</version>
        </dependency>
    </dependencies>
    <executions>
        <execution>
            <phase>generate-sources</phase>
            <goals>
                <goal>wsimport</goal>
            </goals>
            <configuration>
                <wsdlFiles>
                    <wsdlFile>example.wsdl</wsdlFile>
                </wsdlFiles>
                <args>
                    <arg>-B-immutable -B-imm-builder</arg>
                </args>
            </configuration>
        </execution>
    </executions>
</plugin>
```
Next two examples demonstrates the usage of the plugin with CXF *cxf-codegen-plugin* and *cxf-xjc-plugin* mojo.
```xml
<plugin>
    <groupId>org.apache.cxf</groupId>
    <artifactId>cxf-codegen-plugin</artifactId>
    <dependencies>
        <dependency>
            <groupId>com.github.sabomichal</groupId>
            <artifactId>immutable-xjc-plugin</artifactId>
            <version>1.7.0</version>
        </dependency>
    </dependencies>
    <executions>
        <execution>
            <phase>generate-sources</phase>
            <goals>
                <goal>wsdl2java</goal>
            </goals>
            <configuration>
                <wsdlOptions>
                    <wsdlOption>
                        <wsdl>${basedir}/wsdl/example.wsdl</wsdl>
                        <extraargs>
                            <extraarg>-xjc-immutable</extraarg>
                            <extraarg>-xjc-imm-builder</extraarg>
                        </extraargs>
                    </wsdlOption>
                </wsdlOptions>
            </configuration>
        </execution>
    </executions>
</plugin>
```
```xml
<plugin>
    <groupId>org.apache.cxf</groupId>
    <artifactId>cxf-xjc-plugin</artifactId>
    <dependencies>
        <dependency>
            <groupId>com.github.sabomichal</groupId>
            <artifactId>immutable-xjc-plugin</artifactId>
            <version>1.7.0</version>
        </dependency>
    </dependencies>
    <executions>
        <execution>
            <phase>generate-sources</phase>
            <goals>
                <goal>xsd2java</goal>
            </goals>
            <configuration>
                <xsdOptions>
                    <xsdOption>
                        <xsd>${basedir}/wsdl/example.xsd</xsd>
                        <extensionArgs>
                            <arg>-immutable</arg>
                            <arg>-imm-builder</arg>
                        </extensionArgs>
                    </xsdOption>
                </xsdOptions>
            </configuration>
        </execution>
    </executions>
</plugin>
```

#### Gradle
The following example demonstrates the use of the IMMUTABLE-XJC plugin with the Gradle plugin [wsdl2java](https://github.com/nilsmagnus/wsdl2java).
```groovy
plugins {
    id "no.nils.wsdl2java" version "0.12"
}

dependencies {
    wsdl2java 'com.github.sabomichal:immutable-xjc-plugin:1.7.0'
}

wsdl2java {
    wsdlsToGenerate = [
            ['-xjc-immutable', '-xjc-imm-builder', 'src/main/resources/wsdl/example.wsdl']
        ]
    wsdlDir = file("$projectDir/src/main/resources/wsdl")
    cxfVersion = "3.3.5"
    cxfPluginVersion = "3.3.1"
}
```

### Release notes
#### 1.8
* migrated to JAXB 4.0.0 (Jakarta namespace mostly)
* dropped support for java 8
#### 1.7
* added an option to leave all classes non-final
#### 1.6
* added an option to set default values in no-arg constructors
* added an option to generate builder classes that follow the same inheritance hierarchy as their subject classes
* added an option to generate simple builder names
* dropped support for java 6

#### 1.5
* added an option to leave collections mutable
* added an option to generate public constructors only up to n arguments when builder is used

#### 1.4
* added an option to generate non-public constructors
* added an option to generate additional *withAIfNotNull(A a)* builder methods 

#### 1.3
* builder class copy constructor added

#### 1.2
* builder class now contains initialised collection fields
* added generated 'add' methods to incrementally build up the builder collection fields

Initial idea by Milos Kolencik. If you like it, give it a star, if you don't, write an issue.
