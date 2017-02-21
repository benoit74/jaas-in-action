# Introducing JAAS

JAAS, the Java Authentication and Authorization Service, has been a standard part of the Java security framework since version 1.4 version and was available as an optional package in J2SE 1.3. Before that time, the previous security framework provided a way to allow or disallow access to resources based on what code was executing. For example, a class loaded from another location on the Internet would have been considered less trustworthy and disallowed access to the local file system. By changing the security settings, this restriction could be relaxed and a downloaded applet could modify local files. Viewed mainly as a tool for writing clients, early Java didn’t seem to need much more than this.

As the language matured and became popular for server-side applications as well, the existing security framework was deemed too inflexible, so an additional restriction criterion was added:who was running the code. If User A was logged in to the application, file access may be allowed, but not so for User B. Accomplishing this requires authentication and authorization.Authenticationis the process of verifying the identity of users.Authorizationis the process of enforcing access restrictions upon those authenticated users. JAAS is the subsection of the Java security framework that defines how this takes place.

Like most Java APIs, JAAS is exceptionally extensible. Most of the sub-systems in the framework allow substitution of default implementations so that almost any situation can be handled. For example, an application that once stored user ids and passwords in the database can be changed to use Windows OS credentials. Java’s default, file-based mechanism for configuration of access rights can be swapped out with a system that uses a database to store that information. The incredible flexibility of JAAS and the rest of the security architecture, however, produces some complexity. The fact that almost any piece of the entire infrastructure can be overridden or replaced has major implications for coding and configuration. For example, every application server’s JAAS customizations have a different file format for configuring JAAS, all of which are different from the default one provided by Java[^1].

## 

## 

### 



1.4. JAAS

When Java code is executing, it needs to figure out whichPermissions to apply to the current thread of execution. That is, when JAAS is doing aPermissioncheck, it must answer the question “whichPermissions are currently granted?” As mentioned above, JAAS associatesPermissions with a “Subject.”4 In most cases, aSubjectcan be thought of as a “user,” but put more broadly, aSubjectis any entity that Java code is being executed on behalf of. That is, aSubjectneed not always be a person who’s, for example, logged into a system with their username and password.

For example, when a person logs into a JAAS-enabled online banking system, the system creates aSubjectthat represents that user. JAAS resolves whichPermissions theSubjecthas been granted, and associates thosePermissions with theSubject. A “non-human”Subjectcould be another program that is accessing the application. For example, when the nightly batch-processing agent authenticates, or logs into, the system, aSubjectthat represents the agent is created, and the appropriatePermissions are associated with thatSubject.

1.4.1. Authenticating Subjects

So, the first step in JAAS taking effect is logging a user into the system or “authenticating” the users. When a system authenticates a user, it establishes that the user is who they claim to be. As a real world example, when a bank asks one of its clients for their driver’s license and compares the picture to them, they’re authenticating the client’s identity. Similarly, when a user logs in to their bank’s online banking application, they’re prompted to provide a username and password. Because the user knows both of these items, called “credentials,” the online banking system trusts \[believes?\] the claim that they’re Joe User, and allows them to

4 Actually, in the Java security model,Permissions can be associated directly with code as well, further specified by the code’s origin. For example, you could specify that all classes loaded from the JAR somecode.jar be given a specific set ofPermissions. This will be covered in later chapters. For our purposes here, we’ll skip this detail.

This work is licensed under a Creative Commons Attribution-NonCommercial 2.5 License:[http://creativecommons.org/licenses/by-nc/2.5/](http://creativecommons.org/licenses/by-nc/2.5/)

JAAS in Actionby Coté /www.JAASbook.com/www.DrunkAndRetired.com

see their account balances, transfer money, and pay bills.

1.4.2. Principals: Multiple Identities

JAAS doesn’t directly associate a user’s identities with aSubject. Instead, eachSubjectholds onto any number ofPrincipals. In the simplest sense, aPrincipalis an identity. Thus, aSubjectcan be thought of as a container for all ofSubject’s identities, similar to how your wallet contains all of your id cards: driver’s license, social security, insurance card, or pet store club card. For example, aPrincipalcould be:

* The user “jsmith,” which is John Smith’s login for the server.

* Employee number \#4592 which is John Smith’s employee number.

* John’s Social Security number which is used by the HR department.

  Each of these identityPrincipals is associated with John Smith and, thus, once John authenticates with the JAAS-enabled system, eachPrincipalis associated to hisSubject.

  Breaking out aSubject’s identities intoPrincipals creates clear lines of responsibility between the two:Principals tell JAAS who a user is, whileSubjects aggregate the user’s multiple identities. Also, this scheme allows for easier integration with non-JAAS authentication systems, such as single-signon services. For example, when aSubjectis authenticated with a single-signon service, all of the different users are converted toPrincipals, and bundled into theSubject. In this scheme, theSubjectis an umbrella for all the different identities the user, as represented by aSubject, can take on.

  1.4.3. Subjects and Principals: Roles

  In addition toPrincipals representing different identities of aSubject, they can also represent different roles the Subject is authorized to perform. For example, John Smith can perform the following of roles:

* Administer users, including approving new user requests, deleting users, or resetting their passwords.

* Set system-wide configuration properties, like which mail server to use, or the name of the company.

  Each of these roles can be encapsulated in JAAS as aPrincipal. Two roles could be created for the above items: User Administrator, System Administrator. As with identities, rather than directly associate each role’s abilities, or permissions, with aSubject, JAAS associates the role’s abilities withPrincipals. John Smith’sSubject, then, would havePrincipals that represent both of these Administrator roles.

  This work is licensed under a Creative Commons Attribution-NonCommercial 2.5 License:[http://creativecommons.org/licenses/by-nc/2.5/](http://creativecommons.org/licenses/by-nc/2.5/)

JAAS in Actionby Coté /www.JAASbook.com/www.DrunkAndRetired.com

A mapping of Employee and Manager roles to permissions. Note that Chuck is an employee AND a manager.

Separating roles into their ownPrincipals has the same dividing-up-responsibility advantages as separating identities. The primary benefit, however, is with managing and maintaining the user’s permissions. The management of who can do what in these systems can become extremely cumbersome and time consuming if a user’s abilities are directly associated with each Subject instead of a role-basedPrincipal, which is associated with n- users.

For example, suppose we have a system with 5 User Administrator. If we followed a model where permissions were associated directly withSubjects instead ofPrincipals, eachSubjectwould be assigned thePermissionto approve user requests, delete users, and reset their password. During the lifetime of the system, the permissions one of these User Administrators have will change. For example, suppose we decide that resetting a password for a user should only be done by the user: if a User Administrator reset their password, someone other than the user would know the password and might do evil things with that knowledge, or let it slip into the wrong hands. So, we have to modify thePermissionof each of those 5 User Administrators, and remove the reset password permission. Similarly, we must visit each of the 5 User Administrators if we add aPermission.

While this may not seem too onerous a process for 5 users, doing it for more than 5 users can start to be tedious. For example, suppose we had a user base of 10,000 users, 4,032 of which you want to add the new permission “can delete documents.” IfPermissions were directly associated withSubjects, you’d have to visit all 4,032 of those users! Instead, if the 4,032 users are all in the “Document Editor” role, as expressed by aPrincipalin JAAS, you need only edit that one role.

1.4.3. Credentials

In addition toPrincipals,Subjects also haveCredentials. The most common types of credential are a username and password pair. When you log in to your email account, for example, you’re prompted to enter your username and password.

Credentials can take many forms other than a username and password:

This work is licensed under a Creative Commons Attribution-NonCommercial 2.5 License:[http://creativecommons.org/licenses/by-nc/2.5/](http://creativecommons.org/licenses/by-nc/2.5/)

JAAS in Actionby Coté /www.JAASbook.com/www.DrunkAndRetired.com

* A single credential, like your password for a voice mail system.

* A physical credential, like a garage door opener to open your garage.

* A digital certificate.

* A mix of physical and keyed-in credentials, like your ATM card and your PIN

  number.

* Biometric credentials, like your thumbprint or retinal scan.

  Put simply, anything you use to prove your identity is a credential.

  In JAAS,Subjects hold onto two types of credentials: public and private credentials. A username is, in most cases, a public credential: anyone can see your username. A password is, in most cases, a private credential: only the user should know their password. JAAS doesn’t specify an interface, or type, for credentials, as anyObjectcan be a credential. Thus, determining the semantics of a credential—what that credential “means”—are left up to the code that uses the credential.

  1.4.4. Principals and the policy

  Once a Subject has been authenticated, having all of itsPrincipals associated with it, JAAS uses thePolicyservice to resolve whichPermissions thePrincipals are granted. APolicyis simply a singleton that extends the abstract classjava.security.Policy. JAAS uses thePolicy’simplies\(\)andgetPermissions\(\)methods to resolve whichPermissions aSubjecthas been granted.

  The defaultPolicyimplementation is driven by a flat-file, allowing for declaratively configuring thePermissions pre-runtime. Because this implementation is file-driven, however, it’s effectively static: once your application starts, you can’t cleanly change the file’s contents, thus changingPermissions.

  T o provide a more dynamic, runtimePermissionconfiguration, you’ll need to provide your ownPolicyimplementation. You can swap out thePolicyin effect through a VM argument, or at runtime. As with many other JAAS functions, the currently executing code must have permission to swap out thePolicy.

  1.4.5. Access Control: Checking for Permissions

  Once aSubjectis logged in, and has it’sPrincipals associated with it, JAAS can begin to enforce access control. Access control is simply the process of verifying that theSubjecthas been granted any rights required to executing the code. JAAS implements access control by wrappingPermissionchecks around blocks of code. The block of code could be an entire method, or a single line of code. Indeed, for best performance and the finest grain of security control, wrapping the smallest chunk of code possible is the best option.

  Because of it’s nature, then, access control is often done in JAAS in a “try/catch” fashion: attempting to execute the protected code, and then dealing with security exceptions that are thrown due to failedPermissionchecks.Permissionchecking can also be done in a more query-related fashion: before executing a block of code, you can first see if theSubjecthas the appropriate permissions.

  This work is licensed under a Creative Commons Attribution-NonCommercial 2.5 License:[http://creativecommons.org/licenses/by-nc/2.5/](http://creativecommons.org/licenses/by-nc/2.5/)

JAAS in Actionby Coté /www.JAASbook.com/www.DrunkAndRetired.com

Once theSubjectis authenticated, and the appropriatePermissions are loaded, JAAS- enabled code executes securely. Before the code executes, JAAS verifies that theSubjecthas the appropriatePermissions, throwing a security exception if theSubjectdoes not. This is what is meant by JAAS’s claim of being “code centric”: the actual code being protected byPermissions often does the checking itself.

In most cases, the JAAS model of access control requires the code that is performing the sensitive action to do the permission checking itself. For example, instead of blocking access to a callingjava.io.File’s delete\(\)method, the method itself does the security check. This is usually the safest and quickest approach, as finding all the places that calldelete\(\)is much more difficult than simply putting access checks in the method itself. In this code-centric approach, the code must includePermissionchecks, and, thus, be knowable of whichPermissions to check.

In some situations, such a model may be too cumbersome to maintain. Each time a new type ofPermissionis added that’s relevant to deleting a file, you must modify thedelete\(\)method to check for thisPermission. Strategies that use Dynamic Proxies, declarative meta-information \(such as XDoclet or Annotations in J2SE 1.5\) or Aspect Oriented Programming, can be used to more easily solve problems like this. Indeed, those and similar strategies can often be used as a cleaner alternative to embedding access control code yourself.

1.4.6. Pluggability

As the above more code-level talk implies, JAAS is a highly pluggable system. What this means is that you can provide functionality to JAAS that wasn’t originally shipped with the SDK. It also means that you can change the way in which parts of JAAS work. For example, if the default flat-file basedPolicydoesn’t fit your requirements, you can implements and use your own, as later sections in this book will detail.

Unfortunately, as with other high-level frameworks, being pluggable also means there’s a bit of code that you’ll have to write to customize JAAS to your needs. The good news, however, is that youcancustomize it to your needs.

For example, out of the box, JAAS has no idea how to identity users against your HR system. But, you can write a small amount of code that will do just this. Better, instead of having to conform the HR system to how JAAS works, you can customize JAAS to conform to how the HR system works. This is what “pluggable” means in relation to JAAS: you can add in new functionality that wasn’t previously there, and you’re not restricted to the original intent, design, and functionality of the framework.

The authentication system is pluggable primarily through providing customLoginModuleimplementations, while the authorization system is pluggable by providing both customjava.security.Permissionimplementations, andjava.security.Policyimplementations. Chapters X and XX cover creating these custom implementations in great detail.

1.5 Looking ahead

The first part of the book covers the different components of JAAS, going into the above

This work is licensed under a Creative Commons Attribution-NonCommercial 2.5 License:[http://creativecommons.org/licenses/by-nc/2.5/](http://creativecommons.org/licenses/by-nc/2.5/)

JAAS in Actionby Coté /www.JAASbook.com/www.DrunkAndRetired.com

major components and “supporting classes” is much more detail. Included in this part will be an example of creating a database-backedPolicyand customPermissions.

While the first part has many examples of using these JAAS classes, the second part will provide more in-depth examples of common uses of JAAS, such as logging users in, managing user groups, and creating data-centricPermissions.

Finally, the appendixes will go over changes in JAAS in J2SE 5.0, standard J2SEPermissions, and a concise reference for configuring JAAS.

Summary

Our first encounter with security in Java began with the need to provide a secure web application for accessing employee information. With that problem at hand, we started exploring the broad topic of Java security, and narrowed down to the Java Authentication and Authorization Service, or JAAS. We introduced JAAS's primary concepts and classes: permissions, policies, and the service layers needed to enforce the granting of permissions. While we dipped our toe into the code-waters of JAAS, our discussion remained fairly high- level so that we could establish the domain needed to dive into the code.

[^1]: But, we’re Java programmers, we live for that kind of stuff, right?

