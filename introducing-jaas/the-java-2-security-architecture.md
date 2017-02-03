## The Java 2 Security Architecture

The main functionality of the Java 2 security architecture is protecting resources. “Resources” can be anything, but are usually some chunk of data: employee records, databases, or more abstract pieces of data such as class files. The classes in thejava.securitypackage do that work directly by defining the process for testing access permission, and associating permissions with code based on the source from which it was loaded. There are additional subsections and utilities also included in the architecture:

* The Java Cryptography Architecture \(JCA\) defines interfaces and classes for encrypting and decrypting information.

* The Java Secure Socket Extension \(JSSE\) uses the JCA classes to make secure network connections.

* JAAS, the topic of this book, which performs authentication and authorization.

QQQ : image

