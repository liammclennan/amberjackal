Amber Jackal
============

This project is an attempt to build a sane read and write data access library, for relational data, that does not rely on traversing an object graph, collecting state mutations and transforming the result into update scripts. This library is based on the belief that the implicit updates of dirty tracking is inferior to an explicit update strategy. 

A high-level description of the difference is:

ORM style implicit change tracking
-----------

```
Build some objects from data -> make adhoc mutations to those objects -> traverse the object graph and calculate state changes -> push those changes to a datasource
```

Explicit change tracking
-----------

```
Build some objects from data -> explicitly collect required state changes -> push those changes to a datasource
```

For an example of what this looks like, in a document database context, see [SqlDoc](https://github.com/liammclennan/SqlDoc).

API Ideas
=======

Create a session
-------------

```csharp
var session = new AJ.DataSession(typeName => typeName == "Person" ? "People" : typeName + "Table");
```

Insert records
---------------

```csharp
var person = new Person {
                Id = Guid.NewGuid(),
                Name = "Docsesh",
                Age = 90
            };
session.Store(person);

session.Store(new Interest {
                 Id = Guid.NewGuid(),
                 Description = "Playing guitar",
                 PersonId = person.Id
             });
```

Update a record
------------

```csharp
person.Age += 1;
session.Update("Id", person);
```

Delete a record
--------------

```csharp
session.Delete("Id", person.Id);
```

Commit changes
------------

```csharp
using (var connection = new AJ.Connection("Host=myserver;Username=mylogin;Password=mypass;Database=mydatabase")) {
	session.SaveChanges(connection);
} 
```

Queries
-------

### Single record

```csharp
var person = session.Load<Person>("Id", new Guid("BCF52E26-326A-4C90-9D91-DB3FDF60C096"));
```

### Multiple records

```csharp
var adults = session.Query<Person>("select * from People where Age > 18");
```

### Parameterized query

```csharp
int ageLimit = 42;
var people = session.Query<Person>("select * from People where Age < @ageLimit", 
				new Dictionary<string, object> { { "ageLimit", ageLimit } });
```

What data access might look like
==================

```csharp
public PersonWithInterests GetPersonWithInterests(Guid id) {
	var person = session.Load<Person>("Id", id);
	var interests = session.Query<Interest>("select * from Interests where PersonId = @personId", new Dictionary<string,object> {{
		"personId", person.Id
	}});
	return new PersonWithInterests { Person = person, Interests = interests };
}
```
