title: tips of using linqpad
date: 2016-02-19 19:00:27
categories:
- coding
tags:
- .net
- linqpad
---

## linqpad a very convenient tool for quick coding.
You could write sql, c# statements.
And also it's very smart to connect to a database, or table storage.
    context.EntityX.DeleteOnSubmit(`entity`);
    context.EntityX.DeleteAllOnSubmit(`entities`);
    
##
linq pad.
``` cs
var pubs = from p in PublishedNotifications 
           where p.Nid == 1301
		   select p;
PublishedNotifications.DeleteAllOnSubmit(pubs);
SubmitChanges();
```
EF way.
``` cs
using (var context = new MyDbContext())
{
    var itemsToDelete = context.Set<MyTable>();
    context.MyTables.RemoveRange(itemsToDelete);
    context.SaveChanges();
}
```
