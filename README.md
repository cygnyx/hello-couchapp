# hello-couchapp

The couchapp version of Hello, World!
written December 2014 by [cygnyx]

# Introduction
This is a step-by-step guide to creating a 'Hello, World!' pure
couchdb application based on version 1.6.1. The guide is followed by
a general explanation of what is going on.

# Preliminaries
Get [couchdb] running on your
local system and verify that it is operating properly.

Create a file on your local system called 'index.html' with the
following contents:

```
<html>
  <head>
    <title>Hello, World!</title>
  </head>
  <body>
  <h1>Hello, World!</h1>
  </body>
</html>
```

Edit the file '/etc/hosts' and append 'hello' to the line starting
with: '127.0.0.1'. Initially it might look like:

```
127.0.0.1 localhost  
```

And it should be changed to something like:

```
127.0.0.1 localhost hello
```

Make sure the leave a space character between the words. You may need
administrator permission to modify this file.

## Configuration
Using your web browser go to: <http://localhost:5984/_utils>.
This is the couchdb administration page called Futon.
If necessary, sign in with administrator permissions in the lower
right corner of the page.

Near the top left of the page, select 'Create Database ...' and a
 modal dialog box appears with a prompt for the database name.
Enter 'hello' for its name and select 'Create'

Futon will display (the empty) contents of the newly created database.

Near the top left of the page, select 'New Document'.
Near the top right of the page, there is a 'Field' and a 'Source' tab,
select the 'Source' tab instead of 'Field'.
Double click on the text to enable editting.

Replace the default text with:

```
{
  "_id": "_design/app",
  "rewrites": [
       {
           "from": "/",
           "to": "index.html"
       },
       {
           "from": "/index.html",
           "to": "index.html"
       }
  ]
}
```

Near the top left of the page, select 'Save Document'
Futon will redisplay this document with some additional information.

Near the top center of the page, select 'Upload Attachment ...' and a
modal dialog box appears prompting for the location of the local file 
that your created earlier with the name 'index.html'. Choose the
'index.html' file and select 'Upload'.

Futon will redisplay this document again with some additional
information.

On the right side of the page, select 'Configuration' from the menu.

Scroll to the bottom and on the bottom left of the page, select 'Add a
new section' and a modal dialog box appears.
Enter 'vhosts', 'hello', '/hello/_design/app/_rewrite' for section,
option, value, respectively. Select 'Create'

## Verify

In a web browser, goto: <http://hello:5984> to see the 'Hello,
World!' website.

## Explanation

To set up an effective couchapp, or couchdb application, a scheme is
needed to adjust nice looking 'user' URLs into the internally
consistent couchdb URIs. The mechanism used is based on the host name of
the URL. Thereby many hosts can be located on a single couchdb.

The 'hosts' file is a system file that is used to create
translations from system names to IP addresses. The line modified is
the 'loopback' entry that systems use to translate 'localhost' to your
own local system. The value 127.0.0.1 is a special IP value to
always refers to your local system. You can add many aliases for your
local system. These names do not need to have full internet domain
names like 'example.com'. In this case, we use 'hello' as an alias for
the local system. Add an additional alias for each couchapp.
In Unix-like systems the file is located in the '/etc' directory.
In MS Windows the location may depend on your version.

The couchdb includes a system management interface called Futon.
The URI is <http://localhost:5984/_utils>.
Since the hostname 'hello' will be re-routed, it is important to use
'localhost' here.

Initially couchdb is in 'admin party' mode. That is, everyone is an
administrator and can modify the system. You will likely want to
change this behaviour and create an administrator account. If you have
created an administrator account, then you must login with it inorder
to make this configuration changes. The account will logout after a
period of time and it may be necessary to login again.

We create a
database called 'hello' to hold all the data need for the app.
The data for this app consists of the static HTML file to be delivered
and some routing information to translate from nice looking URLs into
couchdb internal URIs.

couchdb manages document oriented databases. The documents are
primarily [JSON] data with some extensions for handling 'attachments'. 
When Futon creates a new document it fills it in with some minimal
JSON which has the _id field. All documents must have this
field. The _ identifies it as a system field. The value of the _id
is a UUID (Universally Unique IDentifier). This default JSON is not
appropriate this part of the app configuration.

couchdb has a number of conventions on how to structure data within
itself.
At some level a database in couchdb is only a set of JSON-like
documents.
The reality is that documents are partitioned into data documents and
design documents. Data documents are the normal run-of-the-mill
documents that contain data. The design documents are important
meta-data documents. Design documents must have ids prefixed with
_design/. Note the _ to identify a system feature and the trailing
/ to identify a hierarchical structure. 

In this case, we create a design document called 'app'. Any other name
could have been used. The fully qualified path to this design document
is  /hello/_design/app. A single database can have many design documents.
In addition, we added 2 rules for
translating URLs into URIs. The structure of these rules is defined by
couchdb. These rules translate / and /index.html to a reference to
index.html. You might notice that couchdb pretty prints the JSON
documents when you save them.

Next the 'index.html' file is 'attached' to this document. In effect,
this includes the file as part of the document. The implementation
is more complex. However, each attachment has a MIME type which
identifies the type of file in addition to the file itself. You can
see that information about the attachment is included in the design
document. A single document can have multiple attachments.

Finally, the app is wired together by adding an entry to the vhosts
configuration. This routes anything with the hostname 'hello' to
/hello/_design/app/_rewrite. The prefix is the fully qualified path
to the design document we created. The suffix _rewrite is a system
identifier that will process URLs and convert them into URIs based on
the rules in the design document.

## Mac OS >10.10 Bonus

The URLs used so far include a port number of 5984.
The default port for HTTP is 80.
To get nice simple URLs we need to get couchdb to process
requests on port 80 instead of 5984.
Similarly, secure requests happen on port 443 instead of 6984.
This change is only needed one time for couchdb, not for each app.

These instructions are only for Mac OS >10.10. But other systems
might have similar configurations.

Create /etc/pf.anchors/org.apache.couchdb with the following contents:

```
rdr pass on lo0 inet proto tcp from any to 127.0.0.1 port 80 -> 127.0.0.1 port 5984
rdr pass on lo0 inet proto tcp from any to 127.0.0.1 port 443 -> 127.0.0.1 port 6984
```

These two lines instruct the system to forward ports 80, 443 to ports 5984, 6984, respectively.

But the port forwarding system needs to know about this file.
Modify /etc/pf.conf to look like:

```
scrub-anchor "com.apple/*"
nat-anchor "com.apple/*"
rdr-anchor "com.apple/*"
rdr-anchor "forwarding"
dummynet-anchor "com.apple/*"
anchor "com.apple/*"
load anchor "com.apple" from "/etc/pf.anchors/com.apple"
load anchor "forwarding" from "/etc/pf.anchors/org.apache.couchdb"
```

I've skipped the comment lines. There are only 2 lines to add -
the lines with "forwarding" in them. The location of the lines is
important. The file contents might differ a bit from this depending
on the system.

Enable the port forwarding system by modifying
/System/Library/LaunchDaemons/com.apple.pfctl.plist
and replace the following 3 lines:

```
		<string>pfctl</string>
		<string>-f</string>
		<string>/etc/pf.conf</string>
```

with:

```
		<string>pfctl</string>
		<string>-ef</string>
		<string>/etc/pf.conf</string>
```

The only change is on the 2nd line, changing `-f` to `-ef`.

This file might be modified during a system update.
If the port forwarding stops working, double check the
contents of this file. This file will start the port forwarding
on each reboot of the system.

To manually start the port forwarding, type:

```
pfctl -ef /etc/pf.conf
```

Now try <http://hello> instead of <http:hello:5894>.
Note that port 5984 and 6894 no longer work.
The links used earlier in this article will no longer work either.

## Next
These steps build a static Hello, World! website.
Adding more complex URL rewriting rules can make documents part of
your website. The principal idea of a couchapp is to provide enough
infrastructure to deliver static files that are needed by
websites to enable dynamic access to the data documents in couchdb.
Since couchdb uses HTTP protocol and JSON documents, it works well
with Ajax based websites.

[cygnyx]: https://github.com/cygnyx
[couchdb]: http://couchdb.apache.org
[JSON]: http://www.json.org
