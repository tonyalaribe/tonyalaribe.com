+++
date = "2017-03-30T18:16:34+01:00"
draft = false
title = "Working with the filesystem in dokku (Persistent File Storage)"
tags = ["golang","dokku"]
+++

Dokku is a self hosted heroku-like platform as a service. And if you've ever used heroku, you're aware of its ephemeral filesystem that deletes all your content with each deployment.  This is a good system, as it forces you to separate your files from your application, and dokku provides an almost identical ephemeral system like heroku.

But most applications need to store files somehow. To go around this, most of us simply upload our files to third party block storage platforms like amazon s3. But how about when we really need to use a file system?

Dokku provides a way to attach a persistent directory to your application, using the Dokku storage plugin. Dokku creates a new directory /var/lib/dokku/data/storage during installation, so its generally accepted that this directory be the base for the mounted storage.

The mounting process is a rather simple process, but first, lets explore.

ssh into you dokku instance, and check out the default dokku data storage directory.

```bash

  cd /var/lib/dokku/data/storage
  ls

```

If this is your first time with dokku persistent storage, the directory should be empty.

Next we create a directory to hold the files for our application
```bash
  mkdir app-name
```

Apps are usually run as the dokku user, so we make sure the dokku user and the 32767 group id has access to the directory.

```bash

chown -R dokku:dokku /var/lib/dokku/data/storage/app-name
chown -R 32767:32767 /var/lib/dokku/data/storage/app-name

```

Then we mount the directory to the /storage directory. You could replace /storage with any directory path, but it should be relative. eg /app/files.

```bash

  dokku storage:mount app-name /var/lib/dokku/data/storage/app-name:/storage

```
What this simply means is that when you write to /storage, you're actually writing to /var/lib/dokku/data/storage/app-name. The best part is that the content of the  /var/lib/dokku/data/storage/app-name directory is persisted between application deployments, since it is outside of the dokku container.

Please be aware that when you destroy your app, the persisted folders will stay on the remote system and you will need to delete them manually if you want them gone.


# Use cases
- I use [bolt db](https://github.com/boltdb/bolt) and [sqlite](https://www.sqlite.org) for some of my web apps, where I dont really want the complexity of a full blown database. So dokku persistent storage helps make these filesystem dependent databases possible.

- I use [bleve search](http://www.blevesearch.com/) to index my data. Most applications I work on, usually need some form of search capabilities, and while most databases like postgresql or mongo db have really good search capabilities, they still remain lacking, with limited tuning capabilities. And bleve on its own, utilizes a key value store which would usually need access to the filesystem for persistence.

- Sometimes I dont want to store uploaded files on heroku. Especially for very quick and  applications.

- Its possible for most applications to be built with a combination of a key value store like bolt, and a full text search engine like bleve. 




For more information on the dokku persistent file storage, visit http://dokku.viewdocs.io/dokku/advanced-usage/persistent-storage/
