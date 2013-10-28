## connection-pool

Generic, efficient, thread safe connection pooling

### Introduction

We needed a safe, efficient way to concurrently access MySQL without connection banging.  

### Features

- Fast
- Thread safe
- Generic (MySQL implementation using Connection/C++ is provided)
- Pre-opens arbitrary number of connections
- Used connections can be returned for immediate reuse
- Unreturned connections are automatically closed and replaced (thanks to shared_ptr reference counting)


### MySQL Example
```cpp
#include <string>
#include <boost/shared_ptr.hpp>
#include "MySQLConnection.h"

// Create a pool of 5 connections
boost::shared_ptr<ConnectionPool<MySQLConnection> > mysql_pool;
shared_ptr<MySQLConnectionFactory>mysql_connection_factory(new MySQLConnectionFactory("mysql_server","mysql_username","mysql_password"));
shared_ptr<ConnectionPool<MySQLConnection> >mysql_pool(new ConnectionPool<MySQLConnection>(5, mysql_connection_factory));


// Get a connection and do something with it
shared_ptr<MySQLConnection> conn=mysql_pool->borrow();		// Throws exception if nothing available

// conn->sql_connection->do_whatever();

// If our code dies here, the connection will be closed and replaced with a new one! :)

// Release
mysql_pool->unborrow(conn);	// Someone else may use this connection now

```

### Design philosophy

We managed to get all of this WITHOUT a separate curator thread.  When you call borrow(), we return the next available connection.  Connections are stored in a std::deque so we can pop from the front and push from back; this makes sure all connections get cycled through as fast as possible (so we don't accidentally hang onto any dead ones for a long time).  If you have a problem with a connection, just let the shared_ptr fall out of scope and the connection will automatically be closed and then replaced with a brand new connection.  In this way, it promotes 'safe' handling of connections without the need to manually ping() (or whatever) each connection periodically.

### Dependencies

Boost
Connection/C++ (for MySQL implementation)