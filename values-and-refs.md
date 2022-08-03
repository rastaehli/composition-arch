### How we access objects
This page is about the basics of identifying and invoking object behavior.

An object has state and behavior that can be invoked through a set of operations.
Every object has a representation, the particular mapping of state and behavior to physical storage, that is understood by the invocation engine.
The state and execution of behavior are considered part of and encapsulated within the object.

The description of operations and expected behavior is not part of the object, but may be documented as an interface.

### Capabilities
Computing systems are composed of many objects, the behavior of one invoking and using the behavior of others.
A capability is a value held (owned by) one object that can be used to invoke behavior of another.

Distinguish object capabilities by their properties:
* values 
  * represent the full state and behavior of an object
  * are immutable - every copy and version behaves the same as the original object
  * often as big as the state of the object, which can be a problem for remote access to something like the library of congress at the start of 2022.
  * to use a value, the holder must understand its representation and lend its execution engine to invoke operations on the value
      * a file may be compressed, or uncompressed in a variety of character encodings
      * an integer may be little-endian or big-endian
      * some representations may identify the encoding, but knowledge of this must still be known by the holder of the value.
* references 
  * are values that uniquely identify an object via some lookup mechanism.  
  * multiple distinct references may identify the same object
  * the reference and its lookup mechanism is part of the holder, not a part of the referenced object.
  * to use a reference the holders invocation engine must resolve the reference, invoke behavior, and receive the response.
    * local invocations may simply transfer the execution context to the target object and unpack the response on return.
    * remote invocations are always delelegated to a local proxy that handles marshalling the remote request and response.
* proxies
  * are intermediary objects (accessed by their own capability) that delegate behavior to a "proxied" object
  * may hide complexities such as security, filtering, or remote access protocols.
  * may adapt the representation and behavioral execution mechanism
* connections
  * are proxies that cache state needed to access a remote object
  * often caches authorization credentials and/or tokens
* names and a lookup service
  * are references that may return zero to many object capabilities from the lookup service
  * are easier for humans to read and remember because they derive from familiar words in common language
  * may be attributes of the object, or only a potential key in the lookup service.

type - a class with potentially many conformant objects
broker - a service that provides objects by type.
snapshot - an immutable copy of a version of an object.  The object may be mutable, but each version is immutable and may be copied by value.

### Communication
A serialization is a representation with bit-serial order and finite length that can be communicated as a snapshot of the object.
The mapping from physical storage to serialization must be understood by the recipient if the snapshot is to be reconstituted as a copy of the original object.

### Composition Requirements
* dependency type declaration
* brokering of capability resolution by type
* dependency capability injection

### Internet naming
* RDF resource description depends on http?
* stable names versus invocable capability URLs
* representation state transfer (REST) model for negotiating the representation of a resource to retrieve
* 