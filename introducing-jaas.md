# Introducing JAAS

JAAS, the Java Authentication and Authorization Service, has been a standard part of the Java security framework since version 1.4 version and was available as an optional package in J2SE 1.3. Before that time, the previous security framework provided a way to allow or disallow access to resources based on what code was executing. For example, a class loaded from another location on the Internet would have been considered less trustworthy and disallowed access to the local file system. By changing the security settings, this restriction could be relaxed and a downloaded applet could modify local files. Viewed mainly as a tool for writing clients, early Java didn’t seem to need much more than this.

As the language matured and became popular for server-side applications as well, the existing security framework was deemed too inflexible, so an additional restriction criterion was added: who was running the code. If User A was logged in to the application, file access may be allowed, but not so for User B. Accomplishing this requires authentication and authorization. Authentication is the process of verifying the identity of users. Authorization is the process of enforcing access restrictions upon those authenticated users. JAAS is the subsection of the Java security framework that defines how this takes place.

Like most Java APIs, JAAS is exceptionally extensible. Most of the sub-systems in the framework allow substitution of default implementations so that almost any situation can be handled. For example, an application that once stored user ids and passwords in the database can be changed to use Windows OS credentials. Java’s default, file-based mechanism for configuration of access rights can be swapped out with a system that uses a database to store that information. The incredible flexibility of JAAS and the rest of the security architecture, however, produces some complexity. The fact that almost any piece of the entire infrastructure can be overridden or replaced has major implications for coding and configuration. For example, every application server’s JAAS customizations have a different file format for configuring JAAS, all of which are different from the default one provided by Java[^1].



[^1]: But, we’re Java programmers, we live for that kind of stuff, right?

