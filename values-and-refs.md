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

Intermediate objects provide loose coupling or other added value to accessing an object
* proxies
  * delegate behavior to a "proxied" object
  * may add/hide complexities such as security, filtering, or remote access protocols.
  * may adapt the representation and behavioral execution mechanism
* connections
  * are proxies that cache state needed to efficiently access a remote object
  * often caches authorization credentials and/or tokens
* names (and a lookup service)
  * are references that may return zero to many object capabilities from the lookup service
  * are easier for humans to read and remember because they derive from familiar words in common language
  * to use a name, the holder must know how to use the lookup service to obtain a capability for the named object
  * a name may or may not be an attribute of the object.
* types (and a broker service)
  * are names that identify the behavior of a class of objects
  * the broker may return one or more object capabilities that conform to a requested type
  * a type may or may not be an attribute of the objects in the class
* attributes (and a lookup service)
  * are behavioral properties of an object
  * can be used to lookup capapabilities for objects with the requested attributes
* meta attributes (and a lookup service)
  * are extra-functional properties of an object, such as physical location or owner
  * can be used to lookup capabilities for objects with the requested meta attributes

### Serialization and Communication
All observed object behavior can be modelled as a sequence of distrete states, each a snapshot of the full representation between invocations.
A serialization is a representation with bit-serial order and finite length that can be communicated as a snapshot of the object.
A serialization is a value, immutable and easily copied, to reconstitute the snapshot in another location.
The mapping from physical storage to serialization must be understood by the recipient if the snapshot is to be reconstituted as a copy of the original object.
A snapshot may be persisted and reconstituted later to allow an object to survive crashes of any one computing environment and migrate as needed to new environments.

### Composition Requirements
A meta-object-protocol for composing objects from dependencies needs to be able to describe:
* what the object's dependencies are in terms of type, attributes, or names.
* where to look for plans for how to resolve a dependency and how to execute those plans.
* how to build the object once dependencies have been provisioned.

### QuA dependency description
To meet the requirements, a QuA Description must include:
* type, or attributes list, or name
* strategy for finding a plan to build the dependency given the specified type, attributes and name.
* strategy for provisioning dependencies of the plan
* strategy for building the dependency according to plan and return a capability
* how to inject capability for this dependency into the parent object

### RDF as a naming scheme
RDF resource description depends provides a strong start for a scalable object naming scheme:
* use simple string and number values to represent immutable property values
* use URIs to identify complex (and possibly mutable) resource subject, relation, and object triples
* use prefix declaration and notation to avoid verbose repetition of long URI prefixes.
* a URI is a name that may (or may not) resolve to a resource via the web.
* HTTP (URI) provides the lookup service to resolve and access a remote resource.
* the URI itself can be used to construct a capability for a remote resource:
  * a REST client to invoke API operations.
  * a request to _GET_ a value _representing_ the resource.

But it's worth mentioning that URIs in general are not the best solution for naming resources.  Problems include:
* broken links when path, protocol, port, server or even domain portions of a URL change.
* the path and URL parameters can be long and unreadable, making them prone to copy and paste errors

#### QuA capabilities
For QuA Names that remain immutable and valid over the lifetime of repositories that contain these references URIs should:
* use only HTTP for the protocol portion to indicate this is a web URI.  Let the HTTP client upgrade the protocol (to HTTPS) if desired.
* use only a stable domain name for the host.  Omit server name to keep this portion short.
* omit the port.  Let the server redirect as needed.
* treat the path as a hierachy of namespaces; each component names an object (or repository) in the parent.

Use QuA Names to identify types and object names. 
Use QuA Names to represent attribute names.  Use strings, numbers, or QuA Names as appropriate for attibute values.
Use QuA Name to identify overriding strategy for finding a plan.  These may be plugin objects found in a local cache so that a capability and invocation of this service is quick and efficient.
Use QuA Name to identify overriding strategy for provisioning.  As above, this may resolve to a local plugin.
Use QuA Name to identify overriding strategy for building.  As above, this may resolve to a local plugin.

Where QuA needs to invoke an object specified by a Description, we must not only build the described object, but return a capability.
The context is responsible for knowing what type of capability is needed and for constructing the capability from an RDF value or QuA Name:
* if a string is expected and the value is a string, it can be used directly
* if a number is expected and the value is a number, it can be used directly
* if an object of type X is expected and the value is a QuA Name, a local resolution engine may return:
  * a local object reference if the QuA Name is found in a local cache
  * a local proxy for a remote X object if a plan can be found to construct such a proxy.
  * a lazy proxy for a remote X object if there is no local plan, but a remote plan might be found and downloaded as needed.
