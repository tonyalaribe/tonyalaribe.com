+++
date = "2017-03-30T18:16:34+01:00"
draft = false
title = "How to use bleve search with a mongo db database in golang"
tags = ["golang","bleve"]
+++

No matter how long you've been using mongo db, at some point, you will become dissatisfied with its data indexing abilities. Almost every dynamic site needs some form of search thee days, but while the built in full text search that comes with mongo db can take you a long way, its just not smart enough for many use cases, like fuzzy searching (miss spelt words and incomplete words), etc.

In this tutorial, I will walk you through how I used [bleve](http://www.blevesearch.com/) to improve the search quality of my already existing golang and mongo db web application (https://calabarpages.com).

The first thing, ofcourse, is to get the bleve package.
```
  go get github.com/blevesearch/bleve
```

For this tutorial, we will use the default bleve backend engine, which relies on bolt db. So, first we will open a bleve index or create an index if non exists. I always make sure to keep just one index instance open, then share that instance all accross the entire app. This is important because the bolt db engine only allows single process access. ie two processes cant read from the same database file at a time (The database is a  physical file). Opening and closing the database/index for each request can result in a lot of load, and errors, since each request would have to wait for every other process using the database to complete.

To share resources across my entire server, I create a config package which holds the shared resources. I use a separate package so the resources can be accessed everywhere, even by other packages within the application.


./config/config.go
```Go
  package config

  import (
    "log"
    "os"

    "github.com/blevesearch/bleve"
  )

  type Conf struct {
    MongoDB     string
    MongoServer string
    Database    *mgo.Database //I also share a mongo db connection accross the app
  	BleveFile   string
  	BleveIndex  bleve.Index
  }

  var config Conf //To hold the shared resources in memory


  func CreateBleveIndex() (bleve.Index, error) {
    /*--------  Content ---*/
  }



  func Init() {
  	MONGOSERVER := os.Getenv("MONGO_URL")

  	session, err := mgo.Dial(MONGOSERVER)
  	if err != nil {
  		log.Println(err)
  	}

  	config = Conf{
  		MongoDB:     MONGODB,
  		MongoServer: MONGOSERVER,
  		Database:    session.DB(MONGODB),
  	}

  	bleveFile := os.Getenv("BLEVE_PATH")
  	if bleveFile == "" {
  		log.Println("Blevefile not set, resulting to default address")
  		bleveFile = "./yellowpages.bleve"
  	}
  	config.BleveFile = bleveFile

  	bleveIndex, err := bleve.Open(bleveFile)
  	if err != nil {
  		log.Println("create bleve index")
  		log.Println(err)
  		bleveIndex, err = CreateBleveIndex()
  	}
  	config.BleveIndex = bleveIndex

  }

  /*Get serves as a way to allow anyone access the single instance of the config information*/
  func Get() *Conf {
  	return &config
  }

```

Note the Init function (not init). This function can be called anywhere, but once. When called, it actually creates the config instance that can be shared across the entire application and accessed with package.Get().

The init function gets the path to the bleve index file, and if it doesn't exist, it creates a new index file.

Now for the interesting part:

```Go
func CreateBleveIndex() (bleve.Index, error) {
	mapping := bleve.NewIndexMapping()
	listingsMapping := bleve.NewDocumentMapping()

	companynameFieldMapping := bleve.NewTextFieldMapping()
	listingsMapping.AddFieldMappingsAt("companyname", companynameFieldMapping)


	mapping.AddDocumentMapping("listing", listingsMapping)

	bIndex, err := bleve.New(config.BleveFile, mapping)
	if err != nil {
		log.Println(err)
		return bIndex, err
	}

	return bIndex, nil
}
```

A bleve index consists of a mapping, which you instruct on how to understand your data.

`mapping := bleve.NewIndexMapping()`

Then you instruct the mapping about the data you will pass into it.  We will be passing in structs of business listings so we create a document mapping to hold the business listing.

`listingsMapping := bleve.NewDocumentMapping()`

Next we describe the most important fields within the listing to improve the indexing. For this, we'll describe just the company name field of the struct, but we could also describe the about field, specialisation, etc.


`companynameFieldMapping := bleve.NewTextFieldMapping()`

We let bleve know that companynameFieldMapping is a mapping under the listing document

`listingsMapping.AddFieldMappingsAt("companyname", companynameFieldMapping)`

In turn, we let bleve know that the listingsMapping is for a document directly under the index.

`mapping.AddDocumentMapping("listing", listingsMapping)`

Finally, we create and return the index

`bIndex, err := bleve.New(config.BleveFile, mapping)
if err != nil {
  log.Println(err)
  return bIndex, err
}`

Now we have an index created, we have to add items to the index.

# Indexing every document in a mongo db collection
If you already had an existing database, you would need to loop through the entire collection and index the data.

```Go

func IndexMongoDBListingsCollectionWithBleve() {
	conf := config.Get()
	mgoSession := conf.Database.Session.Copy()
	defer mgoSession.Close()
	collection := conf.Database.C(config.LISTINGSCOLLECTION).With(mgoSession)

	bleveIndex := config.Get().BleveIndex

	q := collection.Find(bson.M{})
	count, err := q.Count()
	if err != nil {
		log.Println(err)
	}
	cycles := count / 100
  //To prevent using too much memory, we will draw 100 documents from the collection at a time
	for i := 0; i <= cycles; i++ {
		listings := []Listing{}

		err = q.Limit(100).Skip(i * 100).All(&listings)
		if err != nil {
			log.Println(err)
		}

		for _, listing := range listings {
			err = bleveIndex.Index(listing.Slug, listing)
			if err != nil {
				log.Println(err)
				//return err
			}
			log.Printf("indexed %s successfully", listing.Slug)
		}
	}

}
```

The most important part of this code is
`err = bleveIndex.Index(listing.Slug, listing)`

Which indexes a listing, with its slug as the key.

To search, we simply get a list of slugs (the key) that match the search, and get the corresponding documents from mongo.

```Go
func SearchWithIndex(queryString string) (*bleve.SearchResult, error) {
	query := bleve.NewFuzzyQuery(queryString)
	query.Fuzziness = 2
	search := bleve.NewSearchRequest(query)
	bleveIndex := config.Get().BleveIndex

	searchResults, err := bleveIndex.Search(search)
	if err != nil {
		log.Println(err)
		return searchResults, err
	}

	return searchResults, nil
}
```
With SearchWithIndex() we perform a fuzzy search. This simply means that even queries that dont have exact matches will return results that are closest to the query.

```Go
func (r Listing) Search(config *config.Conf, query string) (Listings, error) {
	Results := Listings{}

	mgoSession := config.Database.Session.Copy()
	defer mgoSession.Close()

	collection := config.Database.C("Listings").With(mgoSession)

	searchResult, err := SearchWithIndex(query)
	if err != nil {
		log.Println(err)
	}

  //Create a slice array with all the slugs (keys/IDs)
	documentIdArray := []string{}
	for _, v := range searchResult.Hits {
		documentIdArray = append(documentIdArray, v.ID)
	}

	q := collection.Find(
		bson.M{
			"slug": bson.M{
				"$in": documentIdArray,
			},
		})

	err = q.All(&Results.Data)
	Results.Page = pg
	if err != nil {
		return Results, err
	}
	return Results, nil
}

```

The most important line of note is
```Go
q := collection.Find(
  bson.M{
    "slug": bson.M{
      "$in": documentIdArray,
    },
  })
```

We us the mongo `$in` query to search for all documents with slugs that match the items in the ducmentIdArray.  This way, we dont have to search for the documents one by one.  




Please let me know any improvements I can make to make this tutorial simpler. Also, feel free to let me know any mistakes I might have made, or alternative approaches to solving the challenges the code attempts to solve.
