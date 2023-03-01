# DataBase Management System Implementation Project - Minibase

## Project Summary

Objective of the final phase is to update Minibase to incorporate various functions learnt in lectures as well as integrate the Skyline operation developed in Phase 2 in one of the new functions. We have implemented Clustered BTree Index, Linear Hash Index (Clustered & Unclustered), Group-by/Aggregation operator (based on sorting as well as hash), Index Nested Loop Joins operator, Hash Join operator (hash value based) and Top-K Join operator (NRA based) in Minibase. We have created a driver program to run the given queries on the sample dataset. The queries have been designed to make use of above mentioned operations. Lastly, for all the operations performed, we get the number of disk accesses in the form of read/write counts. Based on the results obtained, a conclusion can be drawn for the operations.

## Index-nested Loop Joins - Individual Contribution

Index Nested Loops Join is used to join two relations using an index on the join attribute of one of the relations. The crux of Index Nested Loops Join (INLJ) logic is similar to that of Nested Loops Join (NLJ) logic, which is to join the tuples/records of outer relation (R) with appropriate tuples of inner relation (S) based on given join condition. However, INLJ differs in the way it accesses the inner relation tuples. Unlike NLJ, where all tuples from inner relation are accessed for every tuple from outer relation, INLJ eliminates the complete access of inner relation tuples for every outer relation tuple by using an appropriate index on inner relation join attribute. Using this index, only the tuples from inner relation that satisfy the join condition ‘θ’ are accessed thereby considerably reducing the cost of join.

Before the INLJ iterator is initialized in the Driver Function, we check for the availability of any index file on the join attribute of the inner relation. In the absence of any desired index file on the desired inner relation field, the join call defaultstoNestedLoopJoin. Incaseofanydesiredindexfileavailabilityonthe desired inner relation field, Index Nested Loop Join is performed based on the order of index file preference. The preference of indices based on different join conditions can be found in the later part of this sub section.

![INLJ Architecture](/Minibase_Implementation/assets/images/INLJ.png)

Once the INLJ iterator has been initialized, the get_next() on this INLJ object will do the following. For every outer relation tuple, an index scan with the join condition (modified by replacing the outer relation symbol of the condition with respective join field values of the current outer relation tuple) is started on the available/preferred join attribute index. The get_next() call on this respective index scan object returns only the inner relation tuples that satisfy the join condition (with respect to the current outer relation tuple’s join attribute value). When this get_next() returns null, the next outer relation tuple is picked and a new index scan with respective join condition values is started. This is continued until the last outer relation tuple and all its inner relation tuples satisfying the condition have been scanned and joined. This indicates the end of the join and the get_next() call on INLJ object (from driver function) returns null.

Although INLJ seems to reduce the total join cost, it comes with an additional cost of ‘index lookup’. This is the cost of accessing the index on the inner relation join attribute to find the target set of inner tuples to be joined with each outer tuple. Therefore, the cost of INLJ for joining an outer relation ‘R’ containing ‘M’ pages of data with ‘X’ number of tuples per page with an inner relation ‘S’ with ‘N’ pages of data with ‘Y’ number of tuples per page will be, 

`Cost of INLJ = M + (M * X) * (ILC)`

Where, ILC -> Index Lookup Cost on inner join attribute index.
It is imperative to note that ‘N’, ‘Y’ and ‘join condition’ all play an implicit role on this overall cost, specific to each join, even though they are not included in the general cost formula. The join condition can be specified by any one of the following five operators.

![Table](/Minibase_Implementation/assets/images/Table.png)

The ‘ILC’ (Index lookup cost) for retrieving inner tuples depends on the type of index available on the join attribute(s) of that relation. The four different types of indices considered in this project and their order of preference based on join condition are listed below.
- Clustered BTree Index,
    - Supports all types of join conditions.
    - Cost is dependent on the height (number of levels) and fan out (Number of pages on each level for a given number of keys) of the tree.
    - Given priority over unclustered BTree index since the cost of range search will be comparatively lesser due to the quasi-sorted data file in case of clustered bTree index.
    - Even though Un-clustered is better in case of equality joins, it is true under the condition that there are no duplicate keys. In case of duplicates, equality search becomes a range search since it has to access all the duplicate keys and these keys are not guaranteed to be on the same datapage with the worst case being no two keys being in the same page. Therefore, a clustered Btree index is preferred over an unclustered BTree index in all the cases.
- Un-clustered BTree Index,
    - Supports all types of join conditions.
    - Expensive than Clustered Btree Index.
    - Cost here is dependent on the height and fanout of the tree as well.
- Clustered Hash Index and
    - Supports only equality joins,
    - Cost of accessing appropriate hash bucket takes constant time,
    - Preferred over all other indices in case of equality join since ‘ILC’ is very less compared to Btree ‘ILC’.
- Unclustered Hash Index.
    - Supports only equality joins,
    - Cost of accessing appropriate hash bucket takes constant time,
    - Preferred over Btree indexes in case of equality join since ‘ILC’ is very less compared to Btree ‘ILC’,
    - Expensive than Clustered Hash Index.
