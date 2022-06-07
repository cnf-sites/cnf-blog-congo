---
title: "Java - Download and Install JDK 1.8 on Windows"
summary: "A detailed step-by-step tutorial on how to download and install jdk 8u172 on Windows."
url: /java-download-install-jdk-8-windows.html
date: 2018-01-09
lastmod: 2018-01-09
tags: ["posts", "java", "jdk"]
draft: false
---

This tutorial has everything you need to know about installing JDK 8 on Windows.

If you're new to Java, I'll show you how to setup the Java Development Kit.

And if you're a Java pro? I'll highlight the needed links that you can use to download the installer.

Bottom line:

If you want to get up and running with Java, **you'll love this tutorial**.

## JDK Download & Install

[Java](https://www.java.com/en/) is a computer programming language that is concurrent, class-based and object-oriented. Java applications compile to bytecode (class file) that can then run on a Java Virtual Machine (JVM).

James Gosling created Java at Sun Microsystems. It is currently owned by the Oracle Corporation.

{{< alert "lightbulb" >}}
Check following guides if you are looking to download and install [JDK 1.5](/java-download-install-jdk-5-windows.html), [JDK 1.6](/java-download-install-jdk-6-windows.html), [JDK 1.7](/java-download-install-jdk-7-windows.html), [JDK 1.9](/java-download-install-jdk-9-windows.html) or [JDK 1.10](/java-download-install-jdk-10-windows.html).
{{< /alert >}}

Java can be obtained from the Oracle Java download page. There are a number of [different Java packages available](https://docs.oracle.com/javaee/6/firstcup/doc/gkhoy.html), for this tutorial we will be installing Java Standard Edition (SE) on Windows.

In order to be able to compile Java code, we need the Java Development Kit (JDK) package that comes with a Java compiler. The JDK package also comes with a Java runtime environment (JRE) that is needed to run compiled Java code.

Scroll to the `Java SE 8u171/ 8u172` section in the middle of the [Oracle Java download page](http://www.oracle.com/technetwork/java/javase/downloads/index.html) and click on the `Download` button right below `JDK`. Then look for the `Java SE Development Kit 8u172` section.

> Here is the direct link to [download the jdk 8u172 installer for Windows 32 or 64 bit](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html).

Accept the License Agreement and pick the correct download for your operating system. In this example, we will use the Windows 64 bit version.

![java 8 download jdk](java-8-download-jdk.png)

Sign in using your Oracle account (or create a new one) and the download should start. Once the download is complete, locate the `jdk-8u172-windows-x64.exe` file and double-click to run the installer.

![java 8 installer start](java-8-installer-start.png)

Click `Next` and on the following screen optionally change the installation location by clicking on the `Change...` button. In this example the default install location of `'C:\Program Files\Java\jdk1.8.0_172\'` was kept. From now on we will refer to this directory as: `[java_install_dir]`.

![java 8 jdk location](java-8-jdk-location.png)

We will not install the public JRE as the JDK Development tools include a private JRE that can run developed code. Select the `Public JRE` dropdown and click on `This feature will not be available.` as shown below.

![java 8 public jre location](java-8-public-jre-location.png)

Click `Next` and then `Close` to finish installing Java.

![java 8 installer finish](java-8-installer-finish.png)

## JDK Configuration

In order for Java applications to be able to run we need to setup a `JAVA_HOME` environment variable that will point to the Java installation directory. In addition, if we want to run Java commands from a command prompt we need to setup the `'PATH'` environment variable to contain the Java bin directory.

When using Windows the above parameters can be configured on the Environment Variables panel. Click on the `Windows Start` button and enter `env` without quotes as shown below.

![edit environment variables](edit-environment-variables.png)

Environment variables can be set at account level or at system level. For this example click on `Edit environment variables for your account` and following panel should appear.

![environment variables](environment-variables.png)

Click on the `New` button and enter `JAVA_HOME` as variable name and the `[java_install_dir]` as variable value. In this tutorial the installation directory is `'C:\Program Files\Java\jdk1.8.0_172'`. Click `OK` to to save.

![java 8 set home](java-8-set-home.png)

Click on the `New` button and enter `PATH` as variable name and `%JAVA_HOME%\bin` as variable value. Click `OK` to save.

> Note that in case a `'PATH'` variable is already present you can add `;%JAVA_HOME%\bin` at the end of the variable value.

![java set path](java-set-path.png)

The result should be as shown below. Click `OK` to close the environment variables panel.

![java 8 environment variables](java-8-environment-variables.png)

In order to test the above configuration, open a command prompt by clicking on the Windows Start button and typing `cmd` followed by pressing `ENTER`. A new command prompt should open in which the following command can be entered to verify the installed Java version:

``` bash
java -version
```

The result should be as shown below.

![java 8 version](java-8-version.png)

This concludes the setting up and configuring JDK 1.8 on Windows.

If you found this post helpful or have any questions or remarks, please leave a comment.
