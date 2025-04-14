# Project 3: PostgreSQL JDBC Driver
PostgreSQL Java Database Connectivity Driver is a driver that helps Java apps talk to PostgreSQL databases.

**Original repository:** https://github.com/pgjdbc/pgjdbc


# Overview of PGJDBC
PGJDBC is an open source Java library that enables Java programs to connect, query and update PostgreSQL databases using the standard SQL.

**Why we chose it?**
It is a tool that is widely used widely in real world productions of Java applications.

This open source library is maintained by The PostgreSQL Global Development Group and contributors from the Java/PostgreSQL community.

# Getting the driver
PostgreSQL Versions: Compatible with PostgreSQL 8.4 and higher, utilizing version 3.0 of the protocol.
The current version of the driver should be compatible with PostgreSQL® 8.2 and higher using the version 3.0 of the PostgreSQL® protocol, and it’s compatible with Java 8 (JDBC 4.2) and above.
Java Versions: Requires Java 8 (JDBC 4.2) or above.

For including the driver in the Maven central we need to add this to pom.xml and replace the latest with the version used:

```
<dependency>
  <groupId>org.postgresql</groupId>
  <artifactId>postgresql</artifactId>
  <version>LATEST</version>
</dependency>
```

# Breadth-Wise analysis
**1. JDBC specifications:** Suppports JDBC 4.2 and above.

**2. Secure connection** using SSL and TLS.

**3. High level architecture of PGJDBC:** It has a modular architecture as follows:
- **Connection Management:** Handles establishing and managing database connections.
- **Query Execution:** Parses and executes SQL queries.
- **Result Handling:** Processes results returned from the database.
- **Type Mapping:** Maps PostgreSQL data types to Java types.
- **Protocol Interface:** Manages communication using PostgreSQL's native protocol.

**4. Files and folders:**

```plaintext
pgjdbc/
├── .github/                # GitHub-specific configurations and workflows
├── benchmarks/             # Performance benchmarking tools
├── build-logic-commons/    # Shared build logic scripts
├── build-logic/            # Build configurations and scripts
├── certdir/                # SSL certificate files for testing
├── config/                 # Configuration files
├── docker/                 # Docker configurations for testing environments
├── docs/                   # Project documentation
├── gradle/                 # Gradle wrapper files
├── packaging/              # Packaging scripts for distributions
├── pgjdbc-osgi-test/       # OSGi-specific tests
├── pgjdbc/                 # Main source code
│   ├── src/
│   │   ├── main/
│   │   │   └── java/
│   │   │       └── org/
│   │   │           └── postgresql/    # Core driver implementation
│   │   └── test/
│   │       └── java/
│   │           └── org/
│   │               └── postgresql/    # Test cases
├── test-anorm-sbt/         # Tests for Anorm (Scala) integration
├── test-gss/               # Tests for GSSAPI (Kerberos) authentication
├── .editorconfig           # Editor configuration
├── .gitignore              # Git ignore rules
├── build.gradle.kts        # Gradle build script
├── settings.gradle.kts     # Gradle settings
├── README.md               # Project overview
```

# Depth Wise analysis
## Core files studied
- **PgConnection.java**
This java file is located at org/postgresql/jdbc/ and manages database connection, transaction management and statement creation.

This file sets up session parameters like autocommit, readOnly, and transactionIsolation, has objects like Statement, PreparedStatement, and CallableStatement.

It also has methods like commit(), rollback(), and setSavepoint() to manage transactions.

- **QueryExecutorImpl.java**
This file located at org/postgresql/core/v3/ handles the execution of SQL queries, parsing and sending to the server and processing the results.

A code snippet we observed:
```public void sendQuery(Query query, ParameterList parameters) throws SQLException {
    // Prepare the query
    SimpleQuery simpleQuery = (SimpleQuery) query;
    // Send the query to the server
    sendParse(simpleQuery, parameters);
    sendBind(simpleQuery, parameters);
    sendExecute(simpleQuery);
    sendSync();
}
```
There are various methods being implemented like sendParse(), sendBind(), sendExecute() which help in parsing, binding and executing the prepared statement.

- **TypeInfoCache.java**
This java file at org/postgresql/core/ caches the information about postgresql data types to optimize type mapping and reduce redundant queries.

This file is responsible for Type OID mapping, SQL type mapping, and array type handling.
```public int getSQLType(String pgTypeName) throws SQLException {
    Integer sqlType = pgTypeNameToSQLType.get(pgTypeName);
    if (sqlType != null) {
        return sqlType;
    }
    // Fallback to querying the database
    int oid = getPGType(pgTypeName);
    return getSQLType(oid);
}
```

    1. pgTypeNameToSQLType: A map caching PostgreSQL type names to SQL types.

    2. getPGType(pgTypeName): Retrieves the OID for the given PostgreSQL type name. 
    
    3. getSQLType(oid): Retrieves the SQL type for the given OID.

- **Key data structures used**
    1. HashMap: Used for caching and quick lookups.
    2. ArrayList: Manages list of parameters and    results.
    3. byte[] Arrays: Handles raw data transmission over sockets.


# Using the driver

1. Firstly any program using JDBC must include: 
```import java.sql.*;```

2. Connecting to the Database:
To connect the following URLs are used:
- jdbc:postgresql:database
- jdbc:postgresql:/
- jdbc:postgresql://host/database
- jdbc:postgresql://host/
- jdbc:postgresql://host:port/database
- jdbc:postgresql://host:port/

3. Parameters for connecting:

```String url = "jdbc:postgresql://localhost/test";
Properties props = new Properties();
props.setProperty("user", "fred");
props.setProperty("password", "secret");
props.setProperty("ssl", "true");
Connection conn = DriverManager.getConnection(url, props);

String url = "jdbc:postgresql://localhost/test?user=fred&password=secret&ssl=true";
Connection conn = DriverManager.getConnection(url);
```
Here user and password are the Strings of credentials.

4. Issuing a query and processing:
To issue any SQL statement at any time you need a PreparedStatement or Statement instance. This example will issue a simple query and print out the first column of each row using a Statement.

```
Statement st = conn.createStatement();
ResultSet rs = st.executeQuery("SELECT * FROM mytable WHERE columnfoo = 500");
while (rs.next()) {
    System.out.print("Column 1 returned ");
    System.out.println(rs.getString(1));
}
rs.close();
st.close();
```

5. Getting the results based on a cursor: Only a small number of rows are fetched using ResultSet on a database cursor as the database can have very large amounts of data.
A small number of rows are cached on the client side of the connection and when exhausted the next block of rows is retrieved by repositioning the cursor.
