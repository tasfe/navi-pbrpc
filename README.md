## Navi-pbrpc
![](https://api.travis-ci.org/neoremind/navi-pbrpc.svg?branch=master)
[![Coverage Status](https://coveralls.io/repos/neoremind/navi-pbrpc/badge.svg)](https://coveralls.io/r/neoremind/navi-pbrpc)
![](https://maven-badges.herokuapp.com/maven-central/com.baidu.beidou/navi-pbrpc/badge.svg)
Navi-pbrpc provides a rpc solution for using protocol buffer. This library enables client and server to communicate in a peer-to-peer and full duplexing way. The server-side is built upon [netty](http://netty.io/) which supports asynchronous and non-blocking io functionality, while the client-side provides a wide variety of options to communicate with server, which includes short live connection, keep-alive tcp connection, high availability and failover strategy.## Quick Start### 1. Prerequisite
Add the below dependency to pom.xml for a maven enabled project.

	<dependency>    	<groupId>com.baidu.beidou</groupId>    	<artifactId>navi-pbrpc</artifactId>    	<version>1.1.0</version>	</dependency>### 2. Make a protobuf generated message
Use `protoc` command to compile an **IDL proto file** and generate a java source file. 
The IDL proto file can define the request and response type. Below is a simple sample:


```package com.baidu.beidou.navi.pbrpc.demo.proto; option cc_generic_services = true;message DemoRequest {    optional int32 user_id = 1;}message DemoResponse {    optional int32 user_id = 1;    optional string user_name = 2;    enum GenderType {        MALE = 1;        FEMALE = 2;    }      optional GenderType gender_type = 3;}
```### 3. Develop server-side service
Develop a server-side service implementation. Below is an example based on the IDL generated java code from the previous step. Note that the method `doSmth` use the generated class `DemoRequest` as the parameter and `DemoResponse` as the return type. The logic here is pretty simple.    public class DemoServiceImpl implements DemoService {        @Override        public DemoResponse doSmth(DemoRequest req) {            DemoResponse.Builder builder = DemoResponse.newBuilder();            builder.setUserId(1);            builder.setUserName("name-1");            builder.setGenderType(DemoResponse.GenderType.MALE);            return builder.build();        }        }
### 4. Expose service and start server

Register the service implementation and set the **service id** as 100. The id here will be used by the client later as a way of locating one specific service.
Then start the server on a port 8088.```PbrpcServer server = new PbrpcServer(8088);server.register(100, new DemoServiceImpl());server.start();```### 5. Develop client to invoke remote service
The framework provides many options to communicate with the server in terms of short live connection or keep-alive connection, high availablity and failover strategy. You can check out more on the [project wiki](http://).Below demostrates how to build a keep-alive connection pool and make a rpc call.
```// 1) Build client with keep-alive pooled connection, and client read timeout is 60sPbrpcClient client = PbrpcClientFactory.buildPooledConnection(new PooledConfiguration(),        "127.0.0.1", 8088, 60000);// 2) Construct request data by using protobufDemoRequest.Builder req = DemoRequest.newBuilder();req.setUserId(1);byte[] data = req.build().toByteArray();// 3) Build message by specifying the service id, and the property provider is used as a client trace sign.PbrpcMsg msg = new PbrpcMsg();msg.setServiceId(100);msg.setProvider("beidou");msg.setData(data);// 4) Asynchronous invocationCallFuture<DemoResponse> future = client.asyncTransport(DemoResponse.class, msg);// 5) Wait response to come. Once rpc call is done, the code will stop blocking right away.DemoResponse res = future.get();// 6) Print out result.System.out.println(res);```
===
### DependenciesNavi-pbrpc tries to leverage minimum amount of dependency libraries, so that project built upon Navi-pbrpc will not be overwhelmed by other unwanted libraries. The following are the dependencies.

    [INFO] +- commons-pool:commons-pool:jar:1.5.7:compile
    [INFO] +- com.google.protobuf:protobuf-java:jar:2.5.0:compile
    [INFO] +- io.netty:netty-all:jar:4.0.28.Final:compile
    [INFO] +- org.javassist:javassist:jar:3.18.1-GA:compile
    [INFO] +- org.slf4j:slf4j-api:jar:1.7.7:compile
    [INFO] +- org.slf4j:slf4j-log4j12:jar:1.7.7:compile
    [INFO] |  \- log4j:log4j:jar:1.2.17:compile
===### More information
Click here to [Tutorials wiki](https://github.com/neoremind/navi-pbrpc/wiki/Tutorials).
### Supports ![](http://neoremind.net/imgs/gmail.png)