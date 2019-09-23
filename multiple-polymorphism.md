## Example 2: Formats and retrieval methods

We have a collection of "entities", say Customers. An entity has a unique ID and a whole bunch of data (name, telephone number, etc). An entity can be represented in a variety of ways. For example, a JSON object representing all of the information about that entity, suitable for populating a web page all about that entity. Or, a smaller JSON object representing a lightweight summary of the entity, suitable for populating a row of a table on a web page. Maybe a simple identifier consisting of almost no information other than the unique ID, suitable for responding to queries like "which customers have names beginning with H".

We want to write an API capable of retrieving entities in any given format. This is not like the last example, since the entities are stored in the database in an abstract form, suitable for representation in any of the desired formats. We are not "converting", only "representing". In this way the problem is easier than in the first example, but it's harder in a different way: we don't want to fetch entities one at a time.

There will be an API endpoint to retrieve a single entity by ID (in any desired format). But there will also be an endpoint to retrieve *all* entities (in a variety of different formats), and endpoints to retrieve sets of entities according to various complex queries: all customers who bought a certain product, all customers living in a certain country, etc. Each one of these queries should support all of the different output formats, at least potentially - we want to completely decouple the problem of determining which entities fulfill a query from the problem of representing entities in a given format.

To see why this is a non-trivial problem, let's consider the "obvious" solution. Create an `Entity` class. For each query, create a procedure to perform the heavy lifting of hitting the network or the hard-drive to collect the relevant entities - `getEntity(int id)`, `getAllEntities()`, etc. In the `Entity` class, write methods `asDetailedJSON()`, `asSummaryJSON()`, for each of the output formats.

The problem with this is performance. If the `Entity` class has to support methods for *all* of the output formats, then it has to contain *all* of the information about an entity. If I want to get a lightweight summary of all of the entities, I'll call `asSummaryJSON()` on each entity returned by `getAllEntities()`. However, although I only need a tiny fraction of the data in each `Entity` object, `getAllEntities()` had no way of knowing I wasn't going to call `asDetailedJSON()`, and so it has no choice but to pack each `Entity` with all of the information I might conceivably want. The trouble is, in order to do this it probably has to make a very expensive I/O call like a `SELECT * FROM entities` SQL query, which is wasteful because `SELECT name, age FROM entities` would have sufficed, and in some scenarios might be an order of magnitude faster.

There's an obvious fix for this problem, which you may have thought of already. Just have `getAllEntities()` retrieve a list of identifiers. The only state in the `Entity` class is then that identifier, and the actual expensive I/O is in the `Entitiy`'s representation methods. When you call `entity.getSummaryJSON()`, it does just enough I/O to get the fields it needs. The problem is that this is again poorly optimized in a different way. Instead of running a single `SELECT name, age FROM entities` call, I now have to run thousands of `SELECT name, age FROM entities WHERE id = :my_id` calls, hitting the network once for every single entity. This is likely to be very slow.

The problem here is that if we don't want our architecture to get in the way of performance, then the operation of "fetching and representing" needs to be treated atomically, expect of course that treating it atomically is terrible architecture, coupling together two subproblems which don't apparently need to be coupled. The best algorithm to use is going to depend on both the query and the desired output format. This can be thought of as a [multiple-polymorphism](multiple-polymorphism.md) problem - which implementation of the `fetchAndRepresent(Query query, OutputFormat format)` procedure to use should depend on the concrete sub-types of *both* of its arguments.

 We can solve this problem by introducing an abstract type (or interface) `Entity` with abstract methods `asDetailedJSON()`, etc. When we call `getAllEntities()`, we receive a list of `UniversalQueryEntity` objects (or something), with an implementation basically like this:

```
class UniversalQueryEntity implements Entity
{
	int entityID;
	EntityDataPool pool;
	
	public UniversalQueryEntity(int entityID, EntityDataPool pool)
	{
		this.entityID = entityID;
		this.pool = pool;
	}

	public EntityDetail asDetailedJSON() {
		return computeDetail(pool.getDetailed());
	}
	
	public EntitySummary asSummaryJSON() {
		return computeSummary(pool.getSummarized());
	}
}

class EntityDataPool
{
	DatabaseResponse data = null;

	public getDetailed()
	{
		if (data == null)
		{
			data = Database.query("SELECT * FROM entities");
		}
		return data;
	}
	
	public getSummarized()
	{
		if (data == null)
		{
			data = Database.query("SELECT name, age FROM entities");
		}
		return data;
	}
}
```

