

Chained functions may be context independent of their proceeding function, or they may be dependent on the results of the proceeding function.  The Chaining Dependency Table below shows the relationship between immediately adjacent, chained functions.  

### Examples
  - `gun.put({hello: "world"}).key("my/data")` 
    - **`key`** proceeded by *`put`* is dependent on the results of *`put`*
    -  i.e. the **`key`** will be assigned to the same object as the *`put`*
  - `gun.put({hello: "world"}).key("my/data").val(function(data){})`
    - **`val`** proceeded by *`key`* is independent of the results of *`key`*, therefore `val` will work its way back to the previous dependent function, in this case the `put` function
    - i.e., both `key` and `val` are dependent on the results of `put`


### Chaining Dependency Table

|        |  .put( )  |  .get( )  |  .path( ) |  .map( )  |  .all( )  |  .key( )  |  .val( )  |  .on( )   |
|:------:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|
| *.put( ).___*  |   D   |***i***|   D   |   D   |***i***|   D   |   D   |   D   |
| *.get( ).___*  |   D   |***i***|   D   |   D   |***i***|   D   |   D   |   D   |
| *.path( ).___* |   D   |***i***|   D   |   D   |***i***|   D   |   D   |   D   |
| *.map( ).___*  |   D   |***i***|   D   |   D   |***i***|   D   |   D   |   D   |
| *.all( ).___*  |***i***|***i***|   D   |   D   |***i***|***i***|   D   |   D   |
| *.key( ).___*  |***i***|***i***|***i***|***i***|***i***|***i***|***i***|***i***|
| *.val( ).___*  |***i***|***i***|***i***|***i***|***i***|***i***|***i***|***i***|
| *.on( ).___*   |***i***|***i***|***i***|***i***|***i***|***i***|***i***|***i***|

Rows represent the initial function, while columns represent the subsequent function.
  - D : Subsequent function is dependent on the previous function.  
  - ***i*** : Subsequent function is independent of the previous function.  