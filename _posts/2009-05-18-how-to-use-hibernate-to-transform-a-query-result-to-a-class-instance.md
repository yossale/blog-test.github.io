---
title: How to use Hibernate to transform a query result to a Class instance

date: 2009-05-18 11:33:00 +0800
categories: [Frameworks, Programming]
tags: [Hibernate, Java]
---

# Problem

Your query returns an object which is a class , but for some reason you can't use it as an entity

Solution:

## The Class

```java
public class MY_CLASS {   
  long a;   
  String b;   
}
```

In the Mapping file:

```sql
<sql-query name="source-settings-by-SOURCEID">   
<return-scalar type="long" column="a" />   
<return-scalar type="string" column="b" />   
<![CDATA[   
  SELECT   
    TableA.a_value as a,   
    TableB.b_value as b   
  FROM TableA, TableB   
  ]]>   
</sql-query>
```

In the call to the query: 

```java
Query query = session.getNamedQuery(QUERY_NAME);   
query.setParameter("PARAM", PARAM); //optional   
rval = query.setResultTransformer(Transformers.aliasToBean(MY_CLASS.class)).list();
```

## Note:

  * The whole point is to keep the column name **exactly** like the class field names , this is how the **aliasToBean** works.
  * In the mapping file ,"long" ,"string" - are lower case on purpose , it failed when I used capitals. (Long, String)