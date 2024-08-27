---
title: "[How2]: Connecting to Firestore (a NoSQL db) with Python!"
weight: 1
tags: ["python", "firestore", "how2", "GCP"]
# author: "Me"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
canonicalURL: "https://mayonesso.com/blog/2024/08/connecting-firestore-with-python/"
disableHLJS: true
disableShare: true
hideSummary: false
summary: "Small guide to connect to Firestore with python"
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: false
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: false
UseHugoToc: true
cover:
    image: "cover.webp"
    relative: false
    hidden: false
---
## • Introduction

Firestore is a NoSQL database that is part of the services that Google offers.

In this article, we will create a connection to this database, taking advantage of the fact that (like other Google products) it has a [free tier](https://cloud.google.com/firestore/pricing).

This post is divided in two main sections:

1.- Creating the database

2.- Obtaining acccess from our project

## • First: Creating the database

The first step is to create a FireStore database in a Google project.

To do this, we visit the [Google Cloud Console](https://console.cloud.google.com/), and on the Databases category select Firestore.

![Google Cloud Services](imgs/1_services.webp)

Here we will create a new database.  
We select the native mode:

![Database setup 1/2](imgs/2_db.webp)

Assign it a name*, and a zone:

![Database setup 2/2](imgs/3_setup.webp)

*If we want to be in the [free tier](https://cloud.google.com/firestore/pricing), we must leave the name of our database to the default value, that is, `(default)`.

Then we select the “Production Rules” mode.

![Database Rules](imgs/4_rules.webp)

And that’s it, we now have our db, and we can start creating our collections and documents:

![Database overview](imgs/5_overview.webp)

Buuuut, what we really want to do is be able to do these types of operations from a Python script located in an external server.

So...

## • Second: Obtaining acccess from our project

To do this, we need to do the following steps!

i.- Install the dependencies in our project  
ii.- Obtain a service account to be able to authenticate correctly  
iii.- Authenticate to Firestore

### i.- Install the dependencies in our project

Here we’ll use `poetry` to install the package `firebase-admin`:

```bash
poetry add firebase-admin
```

And that's it!

### ii.- Obtain a service account to be able to authenticate correctly

Again, we go to the Google Cloud Console, to the IAM & Admin > [Service accounts](https://console.cloud.google.com/projectselector2/iam-admin/serviceaccounts) section and we create a new Service Account:

![Service Account](imgs/6_service_account.webp)

We fill in the details:

![Service Account Details](imgs/7_service_account_details.webp)

Click in `CREATE AND CONTINUE`, and after the Service Account is created, we assign the `Clod Datastore User` role (It seems to be a legacy name, `DataStore` instead of `Firestore`).

![Set Access Role](imgs/8_grant_access.webp)

Other alternatives that also work are: `Firebase Admin SDK Administrator Service Agent` or `Firebase Develop Admin`.  
Note that roles like: `Firestore Service Agent`, or `Firestore Editor` don’t seem to work.

We left the last section as it is;

![Set user access](imgs/9_roles.webp)

Once we click `DONE` our Service Account is ready to be used!

![Service Accounts](imgs/10_service_accounts.webp)

Now, we have to download the file with the credentials that we will use!

As we already can see our new service account is listed, so we click it to see the details.

![Service Account details](imgs/11_service_account_details.webp)

Then navigate to `Keys`, and add a new key in JSON format!

![Keys](imgs/12_keys.webp)

Select `JSON` type and `CREATE`.  
This will create the key and download it to our computer.

![Key creation](imgs/13_key_creation.webp)

### iii.- Authenticate to Firestore

We already have our service account and the corresponding key in a JSON file.  
The next thing is to use this file to authenticate ourselves and be able to interact with our BDD, here is a sample code.

> **Important**; this JSON file is a password (a `key` to be more specific) that can present a security risk in the event of a leak or compromise.  
> It is very important to be careful where we store it, and keep in mind that, since it is a file, it can be mistakenly uploaded to a repository, or to an application that is on the client’s side!

```python
import firebase_admin  
from firebase_admin import credentials  
from firebase_admin import firestore  
  
cred = credentials.Certificate('./path/to/your_service_account.json')  
firebase_admin.initialize_app(cred)  
db = firestore.client()  
  
  
# Write and Append examples:  
  
# Write data with `set`!  
data = {"name": "Los Angeles", "state": "CA", "country": "USA"}  
db.collection("cities").document("LA").set(data)  
  
# Append data with `merge=True`
db.collection("cities").document("BJ").set({"capital": True}, merge=True)  
  
  
# Read data with get():  
doc_ref = db.collection("cities").document("BJ")  
doc = doc_ref.get()  
print(doc.to_dict())
```

</br>
And listo!

----

Useful links:

[Google Docs: Authenticate to Compute Engine](https://cloud.google.com/compute/docs/authentication)  
[Google Docs: Setting up a Service Account](https://cloud.google.com/docs/authentication/provide-credentials-adc#wlif-key)  
[Google Docs: Firestore quickstart](https://firebase.google.com/docs/firestore/quickstart)
