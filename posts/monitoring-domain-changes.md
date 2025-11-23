# Continious Domain Monitoring - Fighting Domain Squatting

![Programmer](https://raw.githubusercontent.com/BarracudaByte/blog/refs/heads/main/images/domain-monitoring.webp?raw=true)

As cyber security analyst it is not uncommon to triage newly registered domains that potentially impersonate your company. There are multiple vendors out there, that are more than happy to notify you for a small amount of money, of a newly registered domain that is potentially domain squatting yours. 

If the domain name indeed sounds very similar but nothing is hosted on the website yet, there is not much that anyone can do. At least until it actually hosts something like a trademarked logo that you can action on.

The problem often though is, once the alert for a newly registered domain comes in, vendors often don't provide a way of keeping an eye on it, in case its content changes and only notify you once it is registered. Luckily for us, we can observe the domain ourselves, with just a few lines of Python.

## How it Works

The idea we are going to implement to check for any domain changes, is pretty straight-forward:
1. The analyst can add a domain to a watchlist
2. A job runs every so often to check all the domains in the watchlist, and gets their DOM
3. The automation compares the DOM to its previous version, and if it has changed it will notify the analyst to review it
4. After the review, the analyst can keep or remove the domain from the watchlist

The interesting part will be comparing the DOM of the websites, as we don't just simply want notify everytime a single letter changed from the content of a site. 

# Implementation

For storing the domains we will be using a SQLite database, as we won't need to setup a server for it and Python has great support for it. We will also only store our domains in there so we know that we won't access it concurrently from other automations. If you are planning on accessing the watchlist from multiple different applications, consider migrating it to a DB like Postgres. 

## Setting up the Database

Python makes working with SQLite database easy and straight-forward and the only thing we need to deal with, is if the DB already exist, if not we create it. There are multiple ways to do this, the easiest is using a simple SQL `CREATE TABLE IF NOT EXIST` syntax. Alternatives would be wrapping everything in some nice `try-except`. 


```python
def setup_db():
    connection = sqlite3.connect("watchlist.db", detect_types = sqlite3.PARSE_DECLTYPES | sqlite3.PARSE_COLNAMES)
    cursor = connection.cursor()
    cursor.execute("CREATE TABLE IF NOT EXISTS domain_watchlist(name TEXT, hash TEXT, created TIMESTAMP, updated TIMESTAMP)")
    cursor.close()
    return connection
```

This function will create and return a connection object that we can then use in the following functions to access our database. The database will be stored in a file called `watchlist.db` in the same folder from where the script will be run. SQLite files can't be directly read as plaintext but you can install for example a VS Code extension to open them. If you want to read more about the SQLite file format check out the [official documentation](https://sqlite.org/fileformat.html).


## Database Functions

We will need in total three functions: one to **add**, to **update** and one to **remove** entires. The database functions are pretty much just SQL statements with a little bit of logic, so I won't go into much detail for them. If you would like to learn more about SQL and writing queries, [W3School](https://www.w3schools.com/sql/) has some good resources.


### Adding Entry to Watchlist

Adding an entry to the database is probably the most complex one, but overall we only want to check if the entry already exist in the database and only add it if not. To check we can use a prepared statement, where we check for the domain name. If it already exist we don't need to add it to our watchlist again and we can simply return. Otherwise we can use the `INSERT INTO` query to add it to our watchlist.


```python

import sqlite3

def add_to_watchlist(connection, domain, dom):
    now = datetime.now()
    deep_hash = ppdeep.hash(dom)
    cursor = connection.cursor()
    # check if the domain is already in the db
    cursor.execute("SELECT * FROM domain_watchlist WHERE name = ?;", (domain, ))
    if cursor.fetchone():
        print(f"[!] The domain '{domain}' is already on the watchlist")
        return
    
    # insert the domain
    insert_query = 'INSERT INTO domain_watchlist VALUES (?, ?, ?, ?);'
    cursor.execute(insert_query, (domain, deep_hash, now, now))
    #cursor.execute("INSERT INTO domain_watchlist VALUES (:domain, :hash, :now, :now)", data)
    connection.commit()
    cursor.close()
    print(f"[+] Successfully added domain '{domain}' to watchlist")
```

### Updating & Deleting Entry in Watchlist

After we have the `add` functionality, `update` and `delete` is rather similar and we only have to adjust our SQL query slightly. 

```python
def update_watchlist(connection, domain, dom):
    now = datetime.now()
    deep_hash = ppdeep.hash(dom)
    cursor = connection.cursor()
    update_statement = 'UPDATE domain_watchlist SET hash = ?, updated = ? WHERE name = ?;'
    cursor.execute(update_statement, (deep_hash, now, domain))
    connection.commit()
    if cursor.rowcount > 0:
        print(f"[+] Successfully updated domain '{domain}' in watchlist")
    cursor.close()
```

Similar for the `delete`, we will use the same code, but use a `DELETE FROM` SQL query instead.

```python
def remove_entry(connection, domain):
    cursor = connection.cursor()
    delete_statement = 'DELETE FROM domain_watchlist WHERE name = ?;'
    cursor.execute(delete_statement, (domain))
    connection.commit()
    if cursor.rowcount > 0:
        print(f"[+] Successfully deleted domain '{domain}' from watchlist")
    cursor.close()
```

## Comparing the DOM

Finally we are getting to the intersting part, the actual logic of our automation! Let's first start with some code to call our previously created functions to setup the DB and add a domain to our watchlist:

```python
import requests # add to imports

# main functionality:
connection = setup_db()
domain = "https://www.barracudabyte.de"
text = requests.get(domain).text
```

In this case we created a connection to our DB, made a request to the domain we want to watch with `requests` and get the `text` which will contain the DOM.

> **Important:** If you are using this automation, I highly recommend using `URLScan.io` or some other application to get the DOM of the website instead of directly quering it! To keep this blog on the main topic we will be using the shortcut of directly requesting it.

Once we have the DOM we can add it to our watchlist with the `add_to_watchlist` function.

### Comparing DOMs with Fuzzy Hashing

Maybe the first thought of how to check if the domain changed would be just to store the full DOM as a `str` in the database, and then doing a string comparison. This is a potential solution but has two major issues. For once, if we have a lot of domains on our watchlist, it may take up a lot of space and if only a single letter of the DOM changes the string comparison would directly be false, not ideal for our case.

The safe space, we could calculate the `MD5` or a `SHA` hash of the content and write that to our DB. This indeed would solve the space issue, but hashing algorithms can create very different hashes with only a small change in the content so the second issue would still persist. 

Instead of just hashing the content, we can use **fuzzy hashing** ([Wikipedia Fuzzy Hashing](https://en.wikipedia.org/wiki/Fuzzy_hashing)). The main idea behind this type of hashing is to detect similartiy of two different items, which is perfect for our use case.

There are different implementations, like `ppdeep`, the pure-Python library for computing context triggered piecewise hashes (CTPH). With `ppdeep` we literally just new a couple more lines of code and our automation is complete. First install `ppdeep` with `pip`:

```python
# run: pip install ppdeep
# add it to your imports
import ppdeep
```

Then we can continue our previous code snippet where we have already queried the DOM of our website. We can now hash the DOM and write it to our DB.

```python
# calculate the hash of the DOM
h = ppdeep.hash(text)

# write it to the watchlist
add_to_watchlist(connection, domain, h)
```

Everytime we want to add a new domain we would do the same steps: query its DOM, calcualte the fuzzy hash of it and then add it to the watchlist. 

### Checking for Changes

Now that we have some content in our database we can have a script that runs every once in a while to check if the content has changed.

```python
# iterate over every entry in the DB
for entry in get_watchlist_entries(connection):
    # query its DOM (replace with URLScan.io or something similar in production!)
    text = requests.get(entry[0]).text

    # get the hash that was stored in the DB
    h1 = entry[1]
    # and calculate the fuzzy hash of the current DOM
    h2 = ppdeep.hash(text)
    try:
        # compare the two hashes
        result = ppdeep.compare(h1, h2)
        print(f"Comparison for {entry[0]}: {result}")
        # check if result is below threshold and notify if yes
    except:
        continue
```

In this snippet we will simply print out the result of the comparison. `ppdeep` will return a number between zero and 100. Where zero means the content is completely different and 100 means that the content is exactly the same. If you run the code for `barracudabyte.de` shortly after each other, you sould see the following output, indicating that the content hasn't changed:

```
Comparison for https://www.barracudabyte.de: 100
```

Depending on how quickly you want to be notified of changes, a threshold of 70 or 80 is often good, so small changes won't trigger a notification.

# Conclusion

Keeping an eye on potential malicious domains in Python is, thanks to libraries like `ppdeep`, simple and straightforward to implement wich only a few lines of code. Similar to any detection engineering work it is important to fine-tune the threshold of when to notify of the domain changes and may be varying depending on the domains that are being watched. 

Fuzzy hashing can of course also be applied to other areas where the comparison of content is the primar concern and it can easily be applied to other automations.