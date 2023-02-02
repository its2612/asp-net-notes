
### Get and Insert Data by using System.Web.Caching package

```C#

public class DataCacheManager
    {

        private Cache _cache;

public DataCacheManager(Cache cache)
        { 
            this._cache = cache; 
        }
//T is the class
public ActionResult cacheGetInser(){

//Get
 IEnumerable<T> cachedData = (IEnumerable<T>)_cache["data_To_Insert_Key"];


//Insert
if(cachedData == null){
cachedData = {Your Data};
_cache.Insert("data_To_Insert_Key"", cachedData , null, DateTime.UtcNow.AddHours(10), Cache.NoSlidingExpiration,
                                                     CacheItemPriority.Normal, null);
}

cachedData = (IEnumerable<T>)_cache["data_To_Insert_Key"];

return view(cachedData);
}

}
```


