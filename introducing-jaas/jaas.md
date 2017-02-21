## 1.3 JAAS

This book is concerned primarily with the lower right box in the diagram above: JAAS. JAAS is a mix of classes specific to JAAS, and classes “borrowed” from other parts of the Java security framework. The primary goal of JAAS is to manage the granting of permissions and performing security checks for those permissions. As such, JAAS is not concerned with other aspects of the Java security framework, such as encryption, digital signatures, or secure connections.

There are several concepts and components that make up JAAS, but all of them revolve around one part: permissions.

### 1.3.2 Permissions

A _permission_ is a named right to perform some action on a target. For example, one permission might be “all classes in the package `com.myappcan` open sockets to the Internet address `www.myapp.com`.”

#### _Who is granted a permission?_

As the example implies, some “entity” is granted a permission. In Java, this entity is usually either a user or a “code base.” Users are a familiar concept, and generally map to a person or process executing code in your application. Users as entities are discussed at length below in the sections on `Subjects` and `Principals`. On the other hand, a code base is a bit more vexing in it’s meaning. A code base is a group of code, usually delineated by a JAR or the URL from which the code was physically loaded. For example, all the classes downloaded as an applet from a remote server could be put into a single code base. Then, because permissions can be applied to code bases, your application could disallow code from that applet from accessing the local file system. You would want to deny access to the local file system to, for example, prevent applets from installing spy- or ad-ware on your machine, or installing viruses. This ability to “sandbox” code, keeping remote, un-trusted code from performing malicious actions, was one of the prime selling points of Java early on.

In Java, sub-classes of the abstract `java.security.Permission` class are used to represent all permissions. There are several types of permissions shipped in the SDK, such as `java.io.FilePermission` for file access, or `java.net.SocketPermission` for network access. The special permission `java.security.AllPermission` serves as a stand-in for any permission. Additionally, you can create any number of custom permissions by extending the class yourself.

A `Permission` is composed of three things, only the first two of which are required:

1. The type of `Permission`, implicit in its class-type.  
2. The `Permission`’s name, generally the target\(s\) the `Permission` controls.
3. Actions that may be performed on the target.

Conceptually, the type and the name of the `Permission` specify what is being accessed. The actions are generally a comma-delimited set of allowed actions. A `FilePermission` named “`pristinebadger.doc`” with actions “`read, write`” would let the possessor read from and write to the file “`pristinebadger.doc`”. The table below illustrates a 4 more examples:

| Permission Type | Target | Action |
| :--- | :--- | :--- |
| ProfilePermission | All | read |
| ProfilePermission | All | write |
| DocumentPermission | "Project Avocado" | read |
| DocumentPermission | All "programming group" documents | write |

The actual “allowing” takes place in the `boolean implies( Permission )` method in `Permission`. When resource access occurs, the resource constructs an instance of the corresponding `Permission` class and passed it to the security framework. The framework then tests if the current security context[^1] has been granted the right\(s\) described by the `Permission` instance. The security framework searches for a permission with the correct type and name. If it finds one, it calls `implies` on it, passing in the newly constructed `Permission` instance. If `true` is returned, access is allowed. The fact that the instance of `Permission` performs the check means that custom permissions can be created. The diagram below illustrates this process:

![](/assets/flow-permission-checking.png)

#### _Permissions are positive_

Permissions express what _can_ be done, not what _cannot_ be done. First, this design decision helps avoids any conflict in permissions. For example, if a user has a permission that gives it access to the C: drive, and another permission that expressly forbids access the C: drive, which one applies? Some method of resolving problems such as this would need to be part of the security model. Instead of devising a conflict resolution process, Java defines only positive permissions. When permissions can only express the positive any chance of conflict is removed.

Alternatively, the creators of JAAS could have chosen to express only negative permissions. A permission would be granted unless it was expressly forbidden. This opens up a security hole in that you must know about everything you want to restrict ahead of time. For example, if a new file enters the system, users will by default have access to that file until the system expressly forbids access. In the mean time, a malicious user could access the file. With a positive permission model, JAAS avoids this need to express everything that is forbidden and the problems that can arise in such a system.

### 1.3.2 The SecurityManager and AccessController

The pre-JAAS security model employed the concept of a security manager, a singleton of `java.lang.SecurityManager` through which all the types of permission were expressed and checked. This class is still used in current versions of Java as the preferred entry point for security checks, but it has traces of the pre-JAAS security model. A quick inspection of the `java.lang.SecurityManager`reveals methods with names like`checkDelete` and `checkExec`. These correspond to each task that needs explicit permission to be performed, such as deleting a file or creating an exec process. Each of these methods will throw a `SecurityException` if the permission in question has not been granted. If the permission has been granted, the method simply returns. The code guarding the resource would call the specific `check` method either granting access, or throwing a `SecurityException` if permission is not granted[^2].

For example, the code in `java.io.File`delete may have included something like:

`public void delete( String filename ) {     
...    
SecurityManager.getInstance().checkDelete(filename) // no SecurityException thrown, so delete file    
...    
}`

If permission has been granted to delete `filename`, the call to `checkDelete` will pass, and the file will be deleted by additional code. If permission has not been granted, `checkDelete` will throw a `SecurityException`, meaning that the code calling delete will need to catch that exception and respond accordingly \(perhaps by displaying an error message to the user\).

A developer wanting to protect a new resource would have to extend the `SecurityManager`, create new `checkXXX` methods, and tell the VM to use this new `SecurityManager` using the runtime property, `java.security.manager`. This unwieldy option spurred the creation of the `Permission` class in Java 2. Also, a new method, `checkPermission`, could now be used to check any `Permission`, enabling the creation of new permissions without having to create a new `SecurityManager`.

To back this new model, a new permission checking service class was introduced `java.security.AccessController`. For backwards compatibility, the existing `SecurityManager` was preserved and, as mentioned above, is still the preferred entry point for permission checking. Like `SecurityManager`, `AccessController` also has a `checkPermission` method. In fact, the default implementation of `SecurityManager` delegates its calls to `AccessController`. This class knows how to access the current thread’s security-related context information, which includes all the permissions allowed for the code-stack that is currently executing. A call to `AccessController.checkPermission()`thus extracts those permission and checks the supplied permission against each piece of code executing on the stack.

#### _Enabling the SecurityManager_

By default, the `SecurityManager`, and thus security as a whole, is disabled when you run Java. The `SecurityManager` can be enabled with the VM argument `-Djava.security.manager`, or by setting the `SecurityManager` to use programmatically at runtime.

### 1.3.3. Policies

Backing the `SecurityManager` and `AccessController` is a policy that expresses which permission are granted to a given security context. Policies map the entities that might attempt to access the resources to theirPermissions. For example, the code found on the D: drive may be disallowed from modifying any files on the C: drive. As we’ll see later in this chapter, this can also be applied to users in addition to code. The main purpose of separating out this concept is that the policy is the logical place for deployment-time configuration. Keeping this responsibility in one place makes managing your security system easier and less error prone.

In the Java security framework, a policy is represented by sub-classes of the abstract class `java.security.Policy`. This class is always a singleton: there is only ever one instance in the VM. Because of this, a `Policy` can be thought of as a service for resolving `Permission` checks. The `Policy` in use can be specified as a VM argument, or can be changed at runtime by calling the static methodPolicy.setPolicy\(policy\). Because the `Policy` being used can be swapped out as needed, custom `Policys` can be used in your Java applications. As we’ll see, coupled with the generic nature of JAAS classes, this allows you to use the Java security framework as a rich user based permission system.

#### _The default policy implementation_

By default, there is no security manager in effect. This effectively allows all permissions for all resources. If the argument        `-Dsecurity.manager=classname` is passed to the VM at runtime, an instance of `classname` is constructed and used. If the property is supplied with no value, a default implementation is chosen.

If the default security manager is specified, the default policy implementation receives its permissions via a flat-file that can be specified at run-time via the VM argument `-Dsecurity.policy.file=policyfilename`. If the path is not specified, the file `$JAVA_HOME/lib/security/java.policy` is used.

![](/assets/sample-grant-clause.png)  


This policy gives all code the ability to write to the file named “`src/conf/cheese.txt`”.

Without this permission an attempt to modify the file will result in a `SecurityException`.

#### _The Default Policy File_

This default policy file is located at `JAVA_HOME/lib/security/java.policy`. This location can be changed with the VM argument `java.security.policy`, which points to the file path of the policy file to use. Also, modifying the` policy.url `properties in the file `JAVA_HOME/lib/security/java.security `will change the location that default policy file is read from.

As we’ll see in later chapters, you can also change the policy being used at runtime, even specifying a class to use to resolve permission checks instead of a flat file.

[^1]: The “security context” includes the rights granted to the currently executing code and, if available, the currently logged in user.

[^2]: This model of security checking is why so many methods in the JDK throwSecurityException.

