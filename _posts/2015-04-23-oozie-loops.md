---
layout: post
title: Loops For Oozie Workflows
tags: [oozie,workflow]
---

The oozie workflow scheduler provides a language for defining workflows as directed acyclic graphs. There is no built in support for loops since loops create circles in the workflow. But in some scenarios it may be helpful to iterate over a list of items with unknown length. While oozie does not offer direct support for loops they can be simulated by recursive calls using a sub-workflow action.

The basic idea is that a workflow calls itself again using a sub-workflow action. I'll illustrate that in a small example. In the example we process a list of files with configurable length. The files need to be specified as input_file_1, input_file_2, ....

### Credits

This is not my own idea. The technique presented here is described in a [post by Robert Kantner on the cdh-users mailing list](https://groups.google.com/a/cloudera.org/forum/#!searchin/cdh-user/oozie$20loop$20subworkflow/cdh-user/j1QQFsz9Z_w/dLXlL8EKnKIJ). 

### Caveat Emptor

Before I start I would like to issue a warning: As it is always the case with recursion you have to take care that your stopping condition for the recursion is implemented correctly. Else you will end up with blocking the whole cluster with an infinite number of recursively created workflow jobs.

As of oozie version 4.1.0 the maximum sub-worfklow depth is limited to 50 as decribed in [OOZIE-1550](https://issues.apache.org/jira/browse/OOZIE-1550).

### Recursive Call

Here we show the action that is responsible for the recursive call. The sub-workflow points to it's own workflow file. To implement a termination condition we increase the variable *counter* by one and pass it as parameter to the recursive call.

```xml
    <action name="loop">
        <sub-workflow>
            <app-path>${wf:appPath()}/../loop.xml</app-path>
            <configuration>
                <property>
                    <name>counter</name>
                    <value>${counter + 1}</value>
                </property>
            </configuration>
        </sub-workflow>
        <ok to="end"/>
        <error to="kill" />
    </action>

```

### Termination

A decision node checks the stopping condition for the recursion. In this example we check if there is an input file for the current *counter* value.

```xml 
    <decision name="check-if-done">
        <switch>
            <case to="process-data">${not empty wf:conf(concat("input_file_", counter))}</case>
            <default to="end" />
        </switch>
    </decision>
```

### Configuration

The list of files can be specified in the workflow configuration file. The following configuration passes a list of three files to our example workflow:


```
input_file_1=myinput1.txt
input_file_2=myinput2.txt
input_file_3=myinput3.txt
```

### Complete Example

Following code shows the complete workflow. It may be called by another workflow that passes in the initial value for the variable *counter*.

```xml 
<workflow-app xmlns="uri:oozie:workflow:0.5'" name="loop-example">

    <parameters>
        <property>
            <name>counter</name>
        </property>
    </parameters>

    <start to="check-if-done"/>

    <decision name="check-if-done">
        <switch>
            <case to="process-data">${not empty wf:conf(concat("input_file_", counter))}</case>
            <default to="end" />
        </switch>
    </decision>

    <action name="process-data">
        <!-- actual locic happens here -->
        <ok to="loop"/>
        <error to="kill" />
    </action>

    <action name="loop">
        <sub-workflow>
            <app-path>${wf:appPath()}/../loop.xml</app-path>
            <configuration>
                <property>
                    <name>counter</name>
                    <value>${counter + 1}</value>
                </property>
            </configuration>
        </sub-workflow>
        <ok to="end"/>
        <error to="kill" />
    </action>


    <kill name="kill">
        <message>Action failed, error
            message[${wf:errorMessage(wf:lastErrorNode())}]
        </message>
    </kill>


    <end name="end" />


</workflow-app>
```