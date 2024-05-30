# openfoodfacts-sqlite-mini
The minified SQLite 3 version of the Open Food Facts database in this repository contains just four columns:

* id
* code
* productName
* countries

There are about 2.45 million records in the SQLite 3 database. I'm calling this derived database a "version" rather than a "copy" of the Open Food Facts database. The SQLite 3 database excludes about 200 columns of data, and also keeps only about 83% of the records from the original database, as explained below. 

On a Mac, the total size on disk is about 160 MB. And yes, this is a **minified** version of the Open Food Facts database.

This repository will consist solely of the SQLite 3 database and this README. 

# OpenFoodFacts 
The database from [OpenFoodFacts.org]([url](https://world.openfoodfacts.org/)) contains over 3 million products. You'll find lots of useful info in the database, including:

* UPC / EAN barcodes
* product names
* country of origin
* ingredients
* nutrition
* ...

Check out the [Wikipedia entry about Open Food Facts]([url](https://en.wikipedia.org/wiki/Open_Food_Facts)).

## OpenFoodFacts database size
As of late May 2024, the Open Food Facts database has over 3 million records. There are 206 columns in the data table.

**An uncompressed CSV copy of the Open Food Facts database is about 9 GB in size.** That's bigger than apps like Excel can typically handle. 

## OpenFoodFacts on GitHub
You'll find numerous repositories and other info by visiting Open Food Facts here on GitHub.

https://github.com/openfoodfacts/

When I checked--briefly--there wasn't a minified SQLite 3 database in GitHub or elsewhere, but the world's a big place. A database derived from the Open Food Facts database must be public, so here we are.

## Justification for the Minified Database
If you need to associate a UPC/EAN code with a product name and the country of origin, but don't (yet) need the ingredients or other information, then this minified SQLite 3 database may be what you need.

There are about 2000 records with duplicate values in the `code` column. Of those, for about 80 codes there are two records that different by the wording in the `productName` column and/or in the listing within the `countries` column. Your SQL queries and/or code to handle that small percentage of cases may different from mine, so I kept the records that have duplicate `code` values.

Of course, you could delete columns and make the minified database yet smaller. Go, you!
 

## Terms of Use: Open Database License (ODbL)
As described on the Open Food Facts [terms of use page]([url](https://world.openfoodfacts.org/terms-of-use)), the Open Food Facts database is available under the [Open Database License (ODbL)]([url](https://opendatacommons.org/licenses/odbl/1-0/)). The same license applies to derivative works, including the minified version of the database provided in this repository.

Some quotes from the Terms of Use page, along with my comments:

> Open Food Facts does not guarantee the accuracy of the information and data present on the site and in the database (included, but not limited to, the product data: photos, barcode, name, generic name, quantity, packaging, brands, categories, origins, labels, certifications, awards, packaging codes, ingredients, additives, allergens, traces, nutrition facts, ecological data etc.).

> The information and data is entered by contributors to the site. It can contain errors due for instance to inaccurate information on labels and packaging, manual input of data, or processing of data.

To be clear: any database this size certainly **will** contain errors. If you include the database in a website, app, or other user-facing software, call attention to the possibility that a particular entry may have errors. An error could simply be a typo.

> Open Food Facts does not guarantee the completeness and comprehensiveness of the information and data present on the site and in the database.

Some records in the database are more complete than others. If you download the entire database, you'll find null/empty cells.

If you try the [Open Food Facts mobile app]([url](https://world.openfoodfacts.org/open-food-facts-mobile-app)), you'll see that there's a lot of data you **could** enter, but that you may not choose to enter.  

## Changes for the Minified Database
The minified version of the Open Food Facts started as a copy, but then I filtered out about 17% of the records. The SQLite 3 database has about 83% of the records of the original database.

A record was removed if either of the following cases was true:

* The length of the `code` column, after removing leading zeroes, was less than 12 characters or more than 14 characters.
* The `productName` column was empty.

Your SQLage may vary, but the SQLite 3 database in this repository contains records only for [UPC / EAN]([url](https://en.wikipedia.org/wiki/Universal_Product_Code)) codes 12 - 14 characters long. That cuts out a lot of records with short and/or incomplete barcode data.

Additional changes:

* `code` column cells: removed leading zeroes (e.g. '000123456789' changed to '123456789')
* column `product_name` renamed to `productName` ('cause I like it like that)

Maybe I introduced some errors, but if so I'll fix those. 

# Programmatic Steps to Create a Minified Database
If you want to create your own version of a minified version of the Open Food Facts database. If you use MongoDB, perhaps you just need the MongoDB dump and you're on your way.

If you're writing code to process the 9 GB CSV data file downloaded from the Open Food Facts database, and if your code will write a CSV output file to review in Excel and/or to import into an SQLite database or other database, then you might follow steps similar to the following:

1. [Download]([url](https://world.openfoodfacts.org/data)) the Open Food Facts database as [JSONL (JSON Lines)]([url](https://jsonlines.org/)), compressed [CSV]([url](https://en.wikipedia.org/wiki/Comma-separated_values)), or [RDF]([url](https://www.w3.org/RDF/)). 
1. (As needed) Uncompress / unzip the downloaded file.
1. In your favorite programming language, create an array of strings to store the minified data you'll output to a CSV file (a file of comma-separated values).
1. Read from the data file one line at a time.
1. Parse the string of the first line, the column names.
1. Split the text using the tab character `\t`. 
1. (Optional) Write the column names to a separate file for reference. Include a 0-based or 1-based index with the column names. There are 206 columns, and you'll want to know the index for each column name.
1. From the collection of substrings created when you split the line of text by `\t`, keep **just** the substrings corresponding to the desired column indices. These will be the column names you're keeping (e.g. ["code", "product_name", "countries"]).
1. Join the substrings with a comma `,` if you intend to create a CSV (comma-separated variable) file.
1. Store the joined string of column names as the first element of your string array.
1. Read the next line in the data file.
1. In the line of text, **replace comma characters `,` with a pipe character `|`** or similar character not otherwise found in the database. (Do not replace the comma character `,` with a semicolon character `;` -- more about that below.)
1. Split the text using the tab character `\t`.
1. From the collection of substrings created by the split, keep **just** the substrings corresponding to the desired column indices. (For example, 0-based indices are 0, 10, and 38 for cells corresponding to the column `code`, `product_name`, and `countries`.)
1. Join the substrings with a comma character `,`.
1. Store the joined string of data cells as the next element of your string array.
1. Repeat steps 11 - 16 above until you reach the end of the raw Open Food Facts data file.
1. (As need) Prepend "\u{FEFF}" to your output file to ensure characters are displayed correctly.
1. Write your array of strings to an output file with the file extension **.csv**. (Remember to append `\n` to each element to add a newline character.)
1. Open your newly created CSV file in Excel, or import the CSV file into a database.
1. Scroll through the data and find out what broke. Fix that.
1. In the database, replace the character `|` with the comma character `,` so that cells in the `ingredients` column and other columns are comma separated again.

Maybe you'll skip those steps and grep | sed | transmogrify your way more elegantly to create a tidy output data file. Good on yer! 

## The raw CSV file from Open Food Facts is tab delimited
As explained inline above, and on the OpenFoodFacts download page, the data provided as a **.csv** file is **not** a traditional file of **C**omma-**S**eparated **V**alues. Instead, the delimiter between fields is a tab character (`\t`).

An `ingredients` cell (0-based column 41) will have ingredients will be deliminated by a comma.

## The temporary replacement character for the comma may depend on your SQLite software
In Terminal on Mac OS, if you import a CSV file, SQLite 3 will interpret both the comma character `,` and the semicolon character `;` as delimiters. If you use the semicolon character `;` as a temporary replacement for the comma, then that could corrrupt records that you import from a CSV file into your database.

Maybe there's a way to configure SQLite 3 to get around that, but I try to keep it simple. 

Here's what I'd enter in Terminal on Mac OS to create a database called "minifyMe.db" and then import the CSV file "output.csv" into a table called "minifiedData":

```
myprompt % sqlite3 minifyMe.db

[sqlite> .import --csv output.csv minifiedData
```

If "output.csv" contains semicolon characters `;`, then in my experience the semi-colons will be treated as field delimiters.

# Other Resources
The [Open Food Facts mobile app]([url](https://world.openfoodfacts.org/open-food-facts-mobile-app)) makes it easy review records in the database.

You only need one or two software packages to work with the SQLite database.

## Open Food Facts mobile app
The mobile app for iPhone or Android is a handy way to scan a product, keep the product in a list, and 

https://world.openfoodfacts.org/open-food-facts-mobile-app

The app allows you to scan a UPC / EAN barcode and see data associated with the product. 

## DB Browser for SQLite
Perhaps because it's the first app I used with SQLite, DB Browser for SQLite remains my go-to interface to review databases, fiddle with tables, and write SQL queries:

https://sqlitebrowser.org/

A quick google search for "sqlite app review" or the like will lead you to apps that you might find more fitting.

## Git Large File Storage (LFS)
The 160 MB file size of "openfoodfacts-sqlite-mini.db" exceeded GitHub's 25 MB limit for browser-based uploads. Installing Git Large File Storage (LFS), following the user guidelines, and then calling `git push` from the command line worked for me.

https://git-lfs.com/

If you create a "derived work" from the Open Food Facts database such as a version of the database in an alternate file format, or minified and filtered as I've done, then you'll need to publish your derived work according to the Terms of Use.

## CSV Files and UTF-8 
If you write output data to a CSV file, you may have to write "\u{FEFF}" to the file before writing the elements of your string array to the file. 

In Swift, you might use something like this:

```
let fileHandle = try FileHandle(forWritingTo: outputUrl)
        
let BOM = "\u{FEFF}"
fileHandle.write(Data(BOM.utf8))

\\loop over remaining elements, and write each element to file: rows[i] + "\n"
```

See 
https://stackoverflow.com/questions/2585024/create-an-utf-8-string-with-bom/2585194#2585194.


## Swift Interface to the SQLite 3 database
If you're going to write Swift code that interfaces to the SQLite3 database in this repository, then I **heartily** recommend using GRDB:

https://github.com/groue/GRDB.swift

The developer is responsive and considerate, the documentation is thorough, and for my Swift projects the GRDB library has worked very well. Of the open source software I've used in the past few years, including both standalone applications and open source libraries, GRDB is my favorite open source project by a wide margin.

For database tasks that would be tricky or infeasible (for me) to implement as SQL queries alone, I use Swift + GRDB.




