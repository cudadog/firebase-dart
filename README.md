# Firebase APIs for Dart VM (server), Fuchsia, and Browser

[![Build Status](https://travis-ci.org/firebase/firebase-dart.svg?branch=master)](https://travis-ci.org/firebase/firebase-dart)

Use [Firebase](https://firebase.google.com)
REST APIs for dart:io apps
(for example: server-side Dart or Fuchsia),
and a Dart wrapper for Firebase's JavaScript API for the browser.

If you are writing Flutter apps for iOS and Android, you should use 
[FlutterFire plugins](https://github.com/flutter/plugins/blob/master/FlutterFire.md)
instead.

You can find more information on how to use Firebase on the
[Getting started](https://firebase.google.com/docs/web/setup) page.

Don't forget to setup correct **rules** for your
[realtime database](https://firebase.google.com/docs/database/security/),
[storage](https://firebase.google.com/docs/storage/security/) and/or 
[firestore](https://firebase.google.com/docs/firestore/security/get-started)
in the Firebase console. 

If you want to use [Firestore](https://firebase.google.com/docs/firestore/quickstart), 
you need to enable it in the Firebase console and include the
[additional js script](#do-you-need-to-use-firestore).

Authentication also has to be enabled in the Firebase console.
For more info, see the
[next section](#before-tests-and-examples-are-run)
in this document.

## Installation

Install the library from pub:

```yaml
dependencies:
  firebase: '^4.3.0'
```

## Using this package with dart:html

### Include Firebase JavaScript source

You must include the original Firebase JavaScript source into your `.html` file
to be able to use the library.

```html
<script src="https://www.gstatic.com/firebasejs/4.10.1/firebase.js"></script>
```

#### Do you need to use Firestore?

Include the `firebase-firestore.js` script also:

```html
<script src="https://www.gstatic.com/firebasejs/4.10.1/firebase.js"></script>
<script src="https://www.gstatic.com/firebasejs/4.10.1/firebase-firestore.js"></script>
```

Firestore library is available in the `firestore.dart` and you can find an example
how to use this library in the [example/firestore](example/firestore).

### Real-time Database Example 

```dart
import 'package:firebase/firebase.dart';

void main() {
  initializeApp(
    apiKey: "YourApiKey",
    authDomain: "YourAuthDomain",
    databaseURL: "YourDatabaseUrl",
    projectId: "YourProjectId",
    storageBucket: "YourStorageBucket");

  Database database = database();
  DatabaseReference ref = database.ref("messages");

  ref.onValue.listen((e) {
    DataSnapshot datasnapshot = e.snapshot;
    // Do something with datasnapshot
  });
}
```

### Firestore Example

```dart
import 'package:firebase/firebase.dart';
import 'package:firebase/firestore.dart' as fs;

void main() {
  initializeApp(
    apiKey: "YourApiKey",
    authDomain: "YourAuthDomain",
    databaseURL: "YourDatabaseUrl",
    projectId: "YourProjectId",
    storageBucket: "YourStorageBucket");

  fs.Firestore firestore = firestore();
  fs.CollectionReference ref = firestore.collection("messages");

  ref.onSnapshot.listen((querySnapshot) {
    querySnapshot.docChanges.forEach((change) {
      if (change.type == "added") {
        // Do something with change.doc
      }     
    });
  });
}
```

## Using this package with dart:io

This library also contains a dart:io client.

Create an instance of `FirebaseClient` and then use the appropriate
method (`GET`, `PUT`, `POST`, `DELETE` or `PATCH`).
More info in the
[official documentation](https://firebase.google.com/docs/reference/rest/database/).

The dart:io client also supports authentication. See the documentation on how to get
[auth credentials](https://firebase.google.com/docs/reference/rest/database/user-auth).

```dart
import 'package:firebase/firebase_io.dart';

void main() {
  var credential = ... // Retrieve auth credential
  var fbClient = new FirebaseClient(credential); // FirebaseClient.anonymous() is also available
  
  var path = ... // Full path to your database location with .json appended
  
  // GET
  var response = await fbClient.get(path);
  
  // DELETE
  await fbClient.delete(path);
  
  ...
}
```

## Examples

You can find more examples on realtime database, auth, storage and firestore in
the [example](https://github.com/firebase/firebase-dart/tree/master/example)
folder.

## Dart Dev Summit 2016 demo app

[Demo app](https://github.com/Janamou/firebase-demo)
which uses Google login, realtime database and storage.

## Before tests and examples are run

You need to ensure a couple of things before tests and examples in this library
are run.

### All tests and examples

Create `config.json` file (see `config.json.sample`) in `lib/src/assets` folder
with configuration for your Firebase project.

To run the io tests, you need to provide the `service_account.json` file. Go to
`Settings/Project settings/Service accounts` tab in your project's Firebase
console, select the `Firebase Admin SDK` and click on the 
`Generate new private key` button, which downloads you a file.
Rename the file to `service_account.json` and put it into the `lib/src/assets`
folder.

> Warning: Use the contents of
[`lib/src/assets`](https://github.com/firebase/firebase-dart/tree/master/lib/src/assets)
is only for development and testing this package.

### App tests

No special action needed here.

### Auth tests and example

Auth tests and some examples need to have **Auth providers** correctly set.
The following providers need to be enabled in Firebase console,
`Auth/Sign-in method` section:

* E-mail/password
* Anonymous
* Phone

### Database tests and example

Database tests and example need to have **public rules** to be able to read and
write to database. Update your rules in Firebase console,
`Database/Realtime Database/Rules` section to:

```json
{
  "rules": {
    ".read": true,
    ".write": true
  }
}
```

> Warning: At the moment, anybody can read and write to your database.
You *usually* don't want to have this in your production apps.
You can find more information on how to setup correct database rules in the 
official
[Firebase documentation](https://firebase.google.com/docs/database/security/). 

### Firestore tests and example

To be able to run tests and example, Firestore needs to be enabled in the 
`Database/Cloud Firestore` section. 

Firestore tests and example need to have **public rules** to be able to read and
write to Firestore. Update your rules in Firebase console,
`Database/Cloud Firestore/Rules` section to:

```
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read, write;
    }
  }
}
```

> Warning: At the moment, anybody can read and write to your Firestore.
You *usually* don't want to have this in your production apps.
You can find more information on how to setup correct Firestore rules in the 
official
[Firebase documentation](https://firebase.google.com/docs/firestore/security/get-started). 

You also need to include the additional `firebase-firestore.js` script.
See [more info](#do-you-need-to-use-firestore).

### Storage tests and example

Storage tests and example need to have **public rules** to be able to read and
write to storage. Update your rules in Firebase console, `Storage/Rules` section
to:

```
service firebase.storage {
  match /b/YOUR_STORAGE_BUCKET_URL/o {
    match /{allPaths=**} {
      allow read, write;
    }
  }
}
```

> Warning: At the moment, anybody can read and write to your storage.
You *usually* don't want to have this in your production apps.
You can find more information on how to setup correct storage rules in the 
official
[Firebase documentation](https://firebase.google.com/docs/storage/security/). 


## Bugs

If you find a bug, please file an
[issue](https://github.com/firebase/firebase-dart/issues).
