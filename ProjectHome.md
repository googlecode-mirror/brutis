## brutis for memcached ##

Brutis is a tool designed to exercise memcache instances by providing reproducible performance data for comparison purposes. Brutis can be useful for sizing memcache clusters as well as testing changes to the system, the hardware, and/or the environment. Much like a dynamometer, the numbers Brutis produces are not as important as the differences in the numbers between changes to the system, the hardware, and/or the environment.

When sizing memcached clusters, Brutis can help by stressing a memcached cluster to see:

  * How many ops the cluster is capable of
  * How much load the network config can take
  * How many connections a memcache cluster is capable of handling (we’ve tested up to 100K connections)

With respect to instance (not cluster) sizing, Brutis can be used to simulate varying object and key sizes to see how many keys/objects can fit in a memcache instance before evictions start.


More information and details can be found in the [Readme](http://code.google.com/p/brutis/wiki/Readme).


Checkout [svn](http://code.google.com/p/brutis/source/checkout) for the latest updates!


---


### Beta 2.0 ###
Added the 2.0 beta to the downloads.. Violin was kind enough to release the code i had been working on at Gear6 for this. Still needs some work and it is just the engine, the web graphing is not in this package.

#### Includes: ####
  * New XML Config files
  * Support for libmemcache & Danga
  * Supports Set/Get/MutliSet/MultiGet/Replace/Delete/Append/Increment/Decrement operations now.


---


If you have any questions, comments, or feedback, Please email me at: zyounker@gmail.com



