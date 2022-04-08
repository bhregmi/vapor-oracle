# vapor-oracle
A sample application showcasing Vapor 4 connecting to an Oracle database using SwiftOracle package.

In this Vapor application, we create a Connection pool with multithreaded option in OCILIB, and then we inject it into Vapor request using StorageKey protocol and an extension of the Application class.  
Each request thread gets its own connection from the pool (already created at the app startup.)   
The connection pool can be configured with a minimum and a maximum number of connections so that each request doesn't have to establish a new database connection.  

The following environment variables should be configured:
- ORACLE_HOME=path_to_instantclient_XX, for example /Users/myuser/instantclient_19_8
- TNS_ADMIN=path_to_a_directory_with_tnsnames.ora_file, for example, /Users/myuser/instantclient_19_8/network/admin/
- LD_LIBRARY_PATH=path_to_instantclient_XX:path_to_OCILIB_libraries, for example: /Users/myuser/instantclient_19_8:/usr/local/lib
- TNS_NAME=database_tns_alias_from_tnsnames.ora
- DATABASE_USER=db_user
- DATABASE_PASSW=db_password
- LOG_LEVEL=debug, trace, info - see Vapor docs



import cocilib // in addition to SwiftOracle

// connection stuff
// ...
try conn.open()
    let cursor = try conn.cursor()
    let value1 = 5 //this is the input code
    var value1var: Int32 = Int32(value1)  // OCILIB call takes a mutable pointer, so need a var
    var value2: Int32 = 0 // value2 is the output code, currently set as 0 to initialize
    let sqlStr = "begin TESTREPORT2(:code, :outputcode); end;" // my test proc returns value1*value1
    
    // using OCILIB manually
    cursor.reset()
    let prepared = OCI_Prepare(cursor.statementPointer, sqlStr)
    assert(prepared == 1)
    
    // bind variables, INOUT mode is the default in OCILIB
    OCI_BindInt(cursor.statementPointer, ":code", &value1var)
    OCI_BindInt(cursor.statementPointer, ":outputcode", &value2)
    
    // execute
    let executed = OCI_Execute(cursor.statementPointer);
    if executed != 1 {
        throw DatabaseErrors.SQLError(DatabaseError(OCI_GetLastError()))
    }
    
    print("outputcode: \(value2)")  //prints out the 0 from above, doesn't display the corresponding code from the db

