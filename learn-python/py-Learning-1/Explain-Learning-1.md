01. The below fix apply when we run the SQL script before the application start.
02. It is not recommended way if we delete the records using the python. 


The error message indicates that the `DELETE` statement on the `product_assignee_table` (the association table for the many-to-many relationship between `Product` and `User`) is attempting to delete more rows than expected. Specifically, it matched 3 rows instead of the expected 1. This suggests a problem with the relationship or the query logic.

---

### Root Cause:
1. **Incorrect Relationship Definition**:
   - The `product.assignees` relationship in the `Product` model might not be properly defined. If the association table does not uniquely identify the relationship between a product and a user, multiple rows could be matched during the `DELETE` operation.

2. **Duplicate Entries in the Association Table**:
   - There might be duplicate rows in the `product_assignee_table`, causing the `DELETE` statement to match multiple rows for the same `product_id` and `assignee_id`.

3. **Session State Issue**:
   - If the `product.assignees.remove(assignee)` operation is called multiple times or the session is not properly managed, it could result in unexpected behavior.

---

### Steps to Resolve:

#### 1. **Check the Relationship Definition**:
   - Verify that the `assignees` relationship in the `Product` model is correctly defined as a many-to-many relationship with a unique constraint in the association table.

   Example:
   ```python
   from app import db

   product_assignee_table = db.Table(
       'product_assignee_table',
       db.Column('product_id', db.Integer, db.ForeignKey('product.id'), primary_key=True),
       db.Column('assignee_id', db.Integer, db.ForeignKey('user.id'), primary_key=True)
   )

   class Product(db.Model):
       __tablename__ = 'product'
       id = db.Column(db.Integer, primary_key=True)
       assignees = db.relationship('User', secondary=product_assignee_table, backref='assigned_products')
   ```

   Ensure that the `product_assignee_table` has a primary key or unique constraint on the combination of `product_id` and `assignee_id`.

#### 2. **Check for Duplicate Rows**:
   - Query the `product_assignee_table` to check for duplicate rows:
     ```sql
     SELECT product_id, assignee_id, COUNT(*)
     FROM product_assignee_table
     GROUP BY product_id, assignee_id
     HAVING COUNT(*) > 1;
     ```
   - If duplicates exist, remove them:
     ```sql
     DELETE FROM product_assignee_table
     WHERE rowid NOT IN (
         SELECT MIN(rowid)
         FROM product_assignee_table
         GROUP BY product_id, assignee_id
     );
     ```

#### 3. **Update the `delete_assignee` Function**:
   - Before removing the assignee, ensure that the user is assigned to the product and handle duplicates gracefully.
   - Here is an updated version of the `delete_assignee` function with improved session management and error handling:

   Updated Function:
   ```python
   def delete_assignee(assignee_id, product_id):
       try:
           assignee = User.query.filter_by(id=assignee_id).first()
           product = Product.query.filter_by(id=product_id).first()

           if not assignee:
               abort(404, description='Assignee not found!')

           if not product:
               abort(404, description='Product not found!')

           # Check if the user is assigned to the product
           if assignee not in product.assignees:
               abort(404, description='Assignee not assigned to this product!')

           # Remove the assignee
           product.assignees.remove(assignee)
           db.session.commit()

           return {'success': True}

       except Exception as e:
           print(e)
           db.session.rollback()
           abort(500, description='An error occurred while unassigning the user.')
        finally:
           db.session.close()
   ```

#### 4. **Test the Fix**:
   - Test the `delete_assignee` function with various scenarios:
     - A valid user assigned to the product.
     - A user not assigned to the product.
     - Invalid `assignee_id` or `product_id`.
---

---

### Debugging Steps:
1. **Check for Other Processes**:
   - Ensure no other processes or threads are accessing the database at the same time. Use tools like `lsof` to check for open file handles:
     ```bash
     lsof | grep your_database.db
     ```

2. **Enable SQLAlchemy Logging**:
   - Enable SQLAlchemy logging to debug the queries being executed:
     ```python
     import logging
     logging.basicConfig()
     logging.getLogger('sqlalchemy.engine').setLevel(logging.INFO)
     ```

3. **Test with a Clean Database**:
   - If the issue persists, test with a clean SQLite database to rule out corruption or other issues.

---

### Additional Notes:
- If the issue persists, inspect the SQLAlchemy-generated SQL queries to ensure they match the expected behavior.
- Ensure that the database schema is consistent with the model definitions, especially for the `product_assignee_table`.

### Long-Term Solution:
If your application requires concurrent access to the database, consider migrating to a more robust database system like PostgreSQL or MySQL. These databases are designed to handle concurrent read and write operations efficiently.

Let me know if you need further assistance!




(sqlite3.OperationalError) database is locked
[SQL: DELETE FROM product_assignee_table WHERE product_assignee_table.product_id = ? AND product_assignee_table.assignee_id = ?]
[parameters: (122, 286)]
(Background on this error at: https://sqlalche.me/e/20/e3q8)
500 Internal Server Error: An error occurred while unassiging the user
INFO:werkzeug:127.0.0.1 - - [20/Apr/2025 14:48:23] "DELETE /api/product/122/assignee/286 HTTP/1.1" 500 -
(sqlite3.OperationalError) database is locked
[SQL: DELETE FROM product_assignee_table WHERE product_assignee_table.product_id = ? AND product_assignee_table.assignee_id = ?]
[parameters: (122, 286)]
(Background on this error at: https://sqlalche.me/e/20/e3q8)
500 Internal Server Error: An error occurred while unassiging the user

GitHub Copilot: The error `(sqlite3.OperationalError) database is locked` occurs when SQLite is unable to acquire a lock on the database file. This typically happens when another process or thread is holding a lock on the database, preventing the current operation (in this case, a `DELETE` statement) from proceeding.

---

### Steps to Resolve:

#### 1. **Ensure Proper Session Management**:
   - Make sure all database sessions are properly committed or rolled back and closed after use. For example:
     ```python
     try:
         product.assignees.remove(assignee)
         db.session.commit()
     except Exception as e:
         db.session.rollback()
         print(e)
         abort(500, description='An error occurred while unassigning the user.')
     finally:
         db.session.close()
     ```

#### 2. **Use a Single Database Connection**:
   - Ensure that your application is not creating multiple connections to the SQLite database. Use a single connection or connection pool managed by SQLAlchemy.

#### 3. **Increase SQLite Timeout**:
   - By default, SQLite has a short timeout for acquiring locks. You can increase the timeout in your SQLAlchemy configuration:
     ```python
     app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///your_database.db?timeout=30'
     ```
   - This sets the timeout to 30 seconds, giving SQLite more time to acquire the lock.

#### 5. **Debug Concurrent Access**:
   - If your application has multiple threads or processes accessing the database, ensure that they are not trying to write to the database simultaneously. SQLite is not designed for high-concurrency write operations.

#### 6. **Switch to a More Robust Database**:
   - If your application requires high levels of concurrency or frequent write operations, consider switching to a more robust database like PostgreSQL or MySQL. SQLite is best suited for lightweight, single-user applications.

---

