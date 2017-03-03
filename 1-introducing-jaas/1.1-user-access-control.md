## User Access Control

Suppose you’re tasked with writing a web application that allows users to log in with an id and password and then allows the users to view their employment information. Because of the sensitivity of the data, it is important that employees not have access to each other’s data. At this point, the protection logic is not very complex: only let the user that is currently logged in see the information that is mapped to himself. But now add the idea of a manager who may be able to see some of the other employees’ items, such as salary or hiring date. Then add the idea of a human resources administrator. Then an accountant. Or an auditor. The CEO. All these users need access to different information and should not be allowed anything more than necessary.

Complex security domains like these are where JAAS comes in handy. Additionally, most application servers and servlet containers use JAAS to provide a way to pass login information into the application. The application can even take advantage of login methods provided by the application server and never deal with the interaction with the user.

By the end of this book, you will both understand and use the functionality in JAAS, and also be able to replace many of the pieces provided by the JDK or whatever application server you may be using with your own custom classes. The rest of this chapter covers high levels security concepts, narrowing down to JAAS at the end.

