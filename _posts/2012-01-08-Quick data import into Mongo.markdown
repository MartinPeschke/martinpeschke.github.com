---
layout: post
title: Quick n dirty data import from relational to mongodb
---

For porting some dusty relational model into MongoDB using python, I was recently scratching my head, trying to figure out how to get table dumps from MySQL straight into Mongo (well, via python/[PyMongo](https://github.com/mongodb/mongo-python-driver)). 

Its actually quite simple and dictionary-like cursors make life even easier:
<code>
import MySQLdb, simplejson
from pymongo import Connection

pymongo_target_collection = Connection('localhost', 27017).testdb.auth_users ### connection.db.collection
pymongo_target_collection.drop()              ### just get rid of the crud, this is a complete LOAD

conn = MySQLdb.connect(host='somehost', user='someuser', passwd='50m3v3ry53cur3pa55w0rd1', db='somedb')
cur = conn.cursor(cursorclass = MySQLdb.cursors.DictCursor)

try:
  sqlselect_stmt = "SELECT * FROM apps.auth_users;"
  cur.execute(sqlselect_stmt)
  
  ## row.copy() purely since the row itself is not really a dictionary and pymongo is checking
  map(pymongo_target_collection.save, [row.copy() for row in cur.fetchall()])
finally: 
  cur.close()
  conn.close()
</code>

If you really need to go by filesystem, dont use CVS if you can help it.

There is the [CVS importer](http://www.mongodb.org/display/DOCS/Import+Export+Tools#ImportExportTools-Example%3AImportfileformat). But that will
* not respect unicode good and proper. Naturally there will be one file imported which will mangle all names. It just always happens.
* It will also loose even minimally complex types, e.g. date types.

Marginally better is using their very own [Mongo Extended Json](http://www.mongodb.org/display/DOCS/Mongo+Extended+JSON), which at least allows dealing with dates.

Just use this little gem
<code>
from pymongo import json_util
</code>
and you get ME(h)JSON for free and simply write that out:
<code>
with open("{}.json".format(INPUT), "w") as outf:
  outf.write(simplejson.dumps(elem, default=json_util.default) + '\n')
</code>

I like it.