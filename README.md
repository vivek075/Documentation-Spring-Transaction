# Documentation-Spring-Transaction
Gives high level details around Transactions.

In Spring Boot, the `@Transactional` annotation is used to manage transactions in a declarative manner. This allows developers to define the scope of a transaction and control its behavior regarding isolation and propagation without manually 
handling the transaction lifecycle.

Understanding `@Transactional`

Basic Usage

The `@Transactional` annotation can be applied to methods or classes to indicate that the method or all methods within the class should be executed within a transaction context.

````
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
public class MyService {

    @Transactional
    public void performDatabaseOperations() {
        // Perform various database operations here
    }
}
````

**Propagation Levels**

Propagation defines how transactions interact with each other. Here are the different propagation levels:

REQUIRED (default): Joins an existing transaction if one exists; otherwise, starts a new one.

REQUIRES_NEW: Suspends the current transaction and starts a new one.

MANDATORY: Joins an existing transaction; throws an exception if no current transaction exists.

NESTED: Executes within a nested transaction if a current transaction exists.

NOT_SUPPORTED: Executes outside of a transaction context; suspends the current transaction if one exists.

NEVER: Executes outside of a transaction context; throws an exception if a current transaction exists.

SUPPORTS: Joins an existing transaction if one exists; executes non-transactionally if none exists.

````
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void newTransactionMethod() {
    // This will run in a new transaction, suspending the current one if it exists
}
````

**Isolation Levels**

Isolation levels define the visibility of changes made in one transaction to other concurrent transactions. Spring supports the following isolation levels:

DEFAULT: Uses the default isolation level of the underlying database.

READ_UNCOMMITTED: Allows dirty reads, non-repeatable reads, and phantom reads.

READ_COMMITTED: Prevents dirty reads; non-repeatable reads and phantom reads can occur.

REPEATABLE_READ: Prevents dirty reads and non-repeatable reads; phantom reads can occur.

SERIALIZABLE: Prevents dirty reads, non-repeatable reads, and phantom reads.

````
@Transactional(isolation = Isolation.SERIALIZABLE)
public void serializableTransactionMethod() {
    // This method will run with the highest isolation level, ensuring full isolation
}
````

**How Transactions Work Internally**
When the `@Transactional` annotation is used, Spring creates a proxy for the annotated method or class. This proxy intercepts method calls and manages the transaction lifecycle, including:

Opening a transaction before the method execution.
Committing the transaction if the method completes successfully.
Rolling back the transaction if an exception occurs (configurable).
Internally, Spring uses a `PlatformTransactionManager` (like `DataSourceTransactionManager` for JDBC) to manage transaction boundaries.

**Difficult Issues in Transactions**
Deadlocks: Occur when two or more transactions are waiting for each other to release resources. Proper handling of lock management and using appropriate isolation levels can mitigate this.

Lost Updates: Occur when multiple transactions read the same data and then update it, with one transaction's update overwriting the other's.

Dirty Reads: One transaction reads uncommitted changes made by another transaction, which might get rolled back later.

Non-repeatable Reads: Data read twice in the same transaction yields different results due to updates by other transactions.

Phantom Reads: New records added by another transaction become visible within the scope of a transaction when it performs a query again.

Nested Transactions: Handling nested transactions can be complex, especially ensuring that inner transactions commit or roll back properly in relation to outer transactions.

````
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Propagation;
import org.springframework.transaction.annotation.Transactional;

@Service
public class OuterService {

    @Autowired
    private InnerService innerService;

    @Transactional
    public void outerMethod() {
        // Perform some database operation
        innerService.innerMethod();
        // Perform another database operation
    }
}

@Service
public class InnerService {

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void innerMethod() {
        // This will run in a new transaction
        // Perform database operation
    }
}
````
In this example, `outerMethod` starts a transaction, and `innerMethod` runs within a new transaction because of the `REQUIRES_NEW` propagation. If `innerMethod` fails, its transaction will be rolled back, but `outerMethod`'s transaction will not be affected.
