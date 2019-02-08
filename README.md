# Json-Response
How to get Json response from Asp.net ( MVC ) to send to Android app or every where that need Json Format of Server Data :)

most of time we need to send data from server in [**Json**](https://www.json.org/) format for example for send data from server by Apis to Android app 
exactly we need Json Format or [XML](https://en.wikipedia.org/wiki/XML) , but i personally use Json most of the time (and i think it's commonly more :) ).

for api coding we can use all server side language like php , [asp.net](https://www.asp.net/) ,...
in this article , my goal is [**Asp.net MVC**](https://www.asp.net/mvc)

in most of databases we have some tables that they are connected to each other [(one-to-one , one-to-many , many-to-many)](https://www.techrepublic.com/article/relational-databases-defining-relationships-between-database-tables/) 
it's mean if we store data in tables that connected to each other , then when fetch one record of that , it has som records that related to itself.

at first imagine we have a table that hasn't any relationship to other tables so if we want to get all data of this in Json format, simply do like bellow

```
List<Users> users = db.Users.ToList();
return Json(users, JsonRequestBehavior.AllowGet);
```

now imagin User table has relationship with other tables , our problem exactly start from here !!! why??
because when this line compiled
```
List<Users> users = db.Users.ToList();
```
**we have all data of users record and ALSO all data that related to each record of User table in other tables !!**
in some conditions it's good (some times that we want to get special record and all related data) and in some conditions it's too bad (some times that we want just independent data of table without any related data)

now if we use this kind return 
```
return Json(users, JsonRequestBehavior.AllowGet);
```
then we had an error that said
`A circular reference was detected while serializing an object of type `

***so what should we do ?***
we can do some things based on WHAT WE WANT !!

***if we want to fetch special fields of base table*** so we can do like bellow 

```
public JsonResult test()
    {
      return Json(
             (from u in db.Users select new {
                 name=u.User_Name , family=u.User_Family , location=u.User_location })
            , JsonRequestBehavior.AllowGet);
    }
```

***if we want to fetch special fields of base table and EVEN related tables*** so we can do like bellow 

```
public JsonResult test()
    {
          return Json(
                     db.Users.Select(u => new {
                                                name = u.User_Name, family = u.User_Family, location=u.User_location
                                               ,gallery = u.Gallery.Select(g => new {
                                                                        galleryName=g.Gallery_Name
                                                                        ,galleryDesc=g.Gallery_Desc
                                           })
                               })
                               , JsonRequestBehavior.AllowGet);
      }
```
****or sth like this****
```
 var gallery = (from u in db.Users 
                        join g in db.Gallery on u.User_ID equals g.Gallery_Admin
                        where u.User_Email == email && g.Gallery_ID == galleryId
                           select new
                        {
                            Gallery_Name = g.Gallery_Name,
                            Gallery_Desc = g.Gallery_Desc,
                            GalleryType = new { GalleryType_ID = g.GalleryType.GalleryType_ID , GalleryType_Title = g.GalleryType.GalleryType_Title, GalleryType_Desc = g.GalleryType.GalleryType_Desc},
                            Gallery_Time = g.Gallery_Time
                        }).FirstOrDefault();
```

***if you want to feth all fields of base table and ALSO all related records in each tables*** so we can do like bellow
```
 public ContentResult test()
    {
            var users = db.Users.ToList();

            var list = JsonConvert.SerializeObject(users,
                                                 Formatting.None,
                                  new JsonSerializerSettings()
                                  {
                                      ReferenceLoopHandling = ReferenceLoopHandling.Ignore
                                  });

            return Content(list, "application/json");
    }
```
