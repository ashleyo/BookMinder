# BookMinder
Some (possibly) helpful sketches for a programming assignment, disorganized, messy, lacking comments

Initial thoughts below. Note the 'initial'. During implementation the initial thought of using TinyCSVParser was abandoned (it seemed hard to read data into value objects it expects reference objects) in favour of some methods found in the VisualBasic namespace.

# L3U16A2#2
## Some design notes and unpacking of the brief for the second part of Unit 16 Assignment 2
### Outline of the brief as given
Read records from a file. You will have to examine the file in order to determine the format, but let us say it is CSV, with one record per line, and string fields such as "Name", "Title" &#8230;, each record representing a book.
Allocate each unique record a unique catalogue identifier and write the file back out again, each record prefixed by its catalogue identifier.

### Notes and Thoughts
As stated the brief is so simplistic that this could almost be one (complicated) line of code. However, we should tackle the project a bit more seriously and consider some practical issues.

1. Reading the CSV. Probably best to use a library for this, there are many. [TinyCSVParser](https://github.com/TinyCsvParser/TinyCsvParser) is one. Reading in a CSV file becomes as easy as 
	```csharp
	TextReader reader =  new  StreamReader("import.txt");  
	var csvReader =  new  CsvReader(reader);  
	var records = csvReader.GetRecords<Automobile>();
	```
2. Note that in this example a Type is provided, `Automobile` in the example, for each record read. The brief does not demand this but it is probably good practice and gives maximum flexibility. So we'll be defining a `Book` type with a bunch of string members such as Name, Title, &#8230;

3. But these objects we are creating, what is the underlying type? A traditional class? Well the snag with that is that class-based objects have reference semantics which means that two objects are only equal if they are the same object. We would want our objects to be the same if they contained the same _values_, that's normally called value semantics.
	We could get that by overriding `ObjectEquals` but then we also need to override `GetHashCode` and should probably do all that by implementing `IEqualityComparer<T>` for our class. Seems like a lot of work.
	
4. C# 10.0 with .Net 6 supports _record_ types which are perfect for our case. They have value semantics and are meant to be used for 'data classes'. However, if we are not entirely up to date, what are our choices? The best, simple plan is probably to use a `struct` rather than a `class`. Structs have value semantics so ... good.

5. Okay, so we are reading our Books into Book struct objects which means stuff like` if (BookA == BookB)`  will work the way we want. Some subtleties that might not occur to you instantly ... Having allocated a catalogue number to a book, _we do not want the book data to mutate_ - because the new data would be inconsistent with the catalogue number. I would strongly recommend making the struct fields readonly.

6. Two (or more) records referring to the same book should get the same catalogue number. A given catalogue number  should only refer to one book. We could allocate identifiers using Net's GUID methods (see [Guid Struct](https://docs.microsoft.com/en-us/dotnet/api/system.guid?view=net-6.0)) but this has two flaws. Getting a GUID twice for the 'same' book would give two GUIDs. Also to know whether a GUID had been used we would have to search all the ones we had already  allocated. Also GUIDs are very unwieldy, example `91952bf6-1561-4a6f-9dfe-55261dc7bd00`
By now you should be thinking hashing. Let's hash all the book data using, for instance,  stuff from the `System.Security.Cryptography` namespace. There is an example [here](https://docs.microsoft.com/en-us/dotnet/standard/security/ensuring-data-integrity-with-hash-codes) although  I should rush to point out that using SHA256 and generating 256 bit hashes is overkill. I would probably use a simple, crytographically weak method that generated shorter hashes (and was faster). And I might not use _all_ of the hash for the catalogue number. It only needs to be long enough to give confidence that a catalogue number collision would not occur. How many books do we have? Make a decision.

7. Using hashes solves our problem. We do not need to worry if we have seen a book before now we are confident same input yields same catalogue code.
8. Job done. Just write the records out again. As CSV would be nice, but the format is not specified. use anything that keeps the fields separate (so not just text!). You could write a `ToString()` override for your struct that outputs CSV directly, why not?  Just be careful with quoting/escaping. CSV has mechanisms for dealing with data fields that contain commas or quotes, make sure you understand them!).   JSON might be a step forward from CSV, [serialize as JSON](https://docs.microsoft.com/en-us/dotnet/standard/serialization/system-text-json-how-to?pivots=dotnet-6-0).


> (c) A P Oliver
> Written with [StackEdit](https://stackedit.io/).
