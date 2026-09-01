# Data Skew 
 - means some partitions have more data than the other partitions
 - so some   executor will take lot of time finish its jobs while other sitting in the idle state.
 - it can lead Out of Memory error
 - it can aslo lead to use costly operation data spill(storing intermediate results into disk while processing)

 # Sort Merge Join 
   ## Shuffle
  - Sort Merge Join shuffles the data using hash(key)%num_of_partitions ,so data with same key goes to same partition does not care about how many partitions we have.
  - Shuffling the data across the network is a costly operation
   ## Sort
  -  Data will be sorted using join key within partitions
   ## Merge
    - for each partition spark moves that two sorted lists in linear format like zipper,when join key is matched then it will merged
   - it increase possiblity of  `data skew ` naturally ,because it shuffles the data using hash formula (same key=same partition)
 # Broad cast Join
   - we can use Broad cast join when one table is very small(maximun size 10 mb) and another table is huge in size.
   - Broad cast join removes costly operations shuffle  instead it copies that small table into all executor memory.
   - it avoids `data skew ` naturally ,because it does not shuffle the data using hash formula
# Adoptive Query Execution
  - adoptive query  execution measures runtime statics
  ## It solves
   ## Shuffle Partition
      - lets say we have default partition size is 200,but data is only have 15 distinct value,so  the remaining 185 partitions going to be empty.
      - so here AQE sees the runtime statics ,so its just creates 15 partition only
   ## Data Skew
     - when  AQE sees uneven partitions ,it will convert the bigger partitions into smaller parts
   ## Sort Merge Join to Broadcast Join
     - even when user write query to do Sort Merge join ,AQE sees one table is very small and another table very big ,so AQE will force spark to do Broadcast join
# Salting and Data Skew
 - salting is the one of technique to reduce data skew.
 - with salting technique,we will add random ness to the  skewed key in table,so while processing the data ,same data  will shuffled to the multiple partitions ,like this data skew promblem can be solved using salting 
 ## when AQE fail to reduce data skew
   - AQE when  data skew ,it will devide the bigger size partition into sub partitions ,even if sub partitions is very bigger compared other partitions ,then it cant solve data skew promblem 70% to 90% percentage.
   - when we aggregations like groupby on skewed key ,AQE cannot solve data skew
 ## salting with join
   ### On big table
  - here we will add new salt column with random numbers within the N numbers.here N means number of partitions
  - we should join by combining key+salt key
  ### On small table
  - here wel will add salt column by exploding array.array contains N numbers.
  - here we do exploding array ,we dont know which row matches from big table to small table
  - explode is costly operation.
# Cache ,Persist and Lazy evaluation
  - cache used  stores intermediate dataframe result,if we using same data frame with same tranformations we can use cache or persisit to avoid recomputations
  - because lazy evalution runs tranformations only when action is called,and follows lineage of tranformations
  - cache stores intermediate dataframe result into memory only
  - in persist we have accessablity to change storage levels like memory only,memory only 2.. 
# Repartition 
- both Repartition and coalesce is used to controll number of output tasks
 - Repartition is used to shuffle the data to create new set partition
 - it uses round robin method or hash partition for distribute the data
 - Repartition can used for increase or decrease partition
 - it tries to distrubute the data evenly
 - data set and tranformation of data set decides wheather we need to use repatition or coalece for decrease output task

# Coalesce 
- Coalesce doet not shuffle the data 
- To decrease output task we can use Coalesce
- coalece does not distrubutes data evenly
- 70 % data  can be go to one file and 30 % of data go to another file 

# Dynamic partition prunning
 - its used in join
 ## Promblem
 - with normal join spark reads all partitions from large table ,join filtering happens only full scan
 ## solution
 - DPP takes matching key from dimensional table ,and filter the large table first using it ,before joining the table
 - so full scann is skipped
## requirement
 - Dynamic partition prunning works only if join key is partition column of large table
 - one side is small enough to broadcast 

# Bucketing
 - Bucketing splits the data into fixed number of buckets based on hashing given bucketing column
 - when two data sets are bucketed on same column and same number of buckets when we do join on them,the shuffling operations will be skipped
 - we do Bucketing on high cardinality column,so splitting data using hashing on high cardinality column will stop small file promblem 
 ## how do decide number of buckets
  - number of buckets=file size/optimal bucket size
  

