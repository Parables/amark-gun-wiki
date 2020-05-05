# Chaining Dependency Table

Chained methods may be context independent of their proceeding method, or they may be dependent on the results of the preceding method. The Chaining Dependency Table below shows the relationship between immediately adjacent, chained functions.

### Examples
  - `gun.put({ hello: "world" }).key("my/data")`
    - **`key`** proceeded by *`put`* is dependent on the results of *`put`*
    -  i.e. the **`key`** will be assigned to the same object as the *`put`*
  - `gun.put({ hello: "world" }).key("my/data").val(function (data) {})`
    - **`val`** proceeded by *`key`* is independent of the results of *`key`*, therefore `val` will work its way back to the previous dependent method, in this case the `put` method
    - i.e., both `key` and `val` are dependent on the results of `put`

|                |  .put( )  |  .get( )  |  .path( ) |  .map( )  |  .key( )  |  .val( )  |  .on( )   |
|:--------------:|:---------:|:---------:|:---------:|:---------:|:---------:|:---------:|:---------:|
| *.put( ).___*  |     D     |*****i*****|     D     |     D     |     D     |     D     |     D     |
| *.get( ).___*  |     D     |*****i*****|     D     |     D     |     D     |     D     |     D     |
| *.path( ).___* |     D     |*****i*****|     D     |     D     |     D     |     D     |     D     |
| *.map( ).___*  |     D     |*****i*****|     D     |     D     |     D     |     D     |     D     |
| *.key( ).___*  |*****i*****|*****i*****|*****i*****|*****i*****|*****i*****|*****i*****|*****i*****|
| *.val( ).___*  |*****i*****|*****i*****|*****i*****|*****i*****|*****i*****|*****i*****|*****i*****|
| *.on( ).___*   |*****i*****|*****i*****|*****i*****|*****i*****|*****i*****|*****i*****|*****i*****|
| *.not( ).___*  |     D     |*****i*****|     D     |     D     |     D     |     D     |     D     |

Rows represent the initial method, while columns represent the subsequent method.
  - D : Subsequent method is dependent on the previous method.  
  - ***i*** : Subsequent method is independent of the previous method.  