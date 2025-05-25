Used the below method to delete the all records which having a duplicate entry. But there is isue in the code. 

Issue:
```
Class 'sqlalchemy.engine.row.Row' is not mapped
500 Internal Server Error: An error occurred while unassiging the user
```

Issue Description:
The error Class 'sqlalchemy.engine.row.Row' is not mapped occurs because the query is returning raw rows (instances of sqlalchemy.engine.row.Row) instead of mapped ORM objects. This happens when querying a Table object directly, as in the case of product_assignee_table, which is likely defined as a SQLAlchemy Table object and not an ORM model.

```python
def delete_assignee(user_id, product_id):
    try:
        assignee = User.query.filter_by(id=user_id).first()
        product = Product.query.filter_by(id=product_id).first()

        if not assignee:
            abort(404, description='Assignee not found!')

        if not product:
            abort(404, description='Product not found!')

        if assignee not in product.assignees:
            abort(404, description='Assignee not found in product!')

        # Debugging: Print the schema of product_assignee_table
        print(product_assignee_table.columns.keys())

        # Query the association table and delete the specific row
        product_assginee_entries = db.session.query(product_assignee_table).filter_by(
            product_id=product.id, assignee_id=assignee.id
        ).all()

        if not product_assginee_entries:
            abort(404, description='No matching row found to delete.')

        # Explicitly delete the row from the association table
        for product_assginee_entry in product_assginee_entries:
            db.session.delete(product_assginee_entry)

        db.session.commit()

        return {'success': True}

    except Exception as e:
        print(e)
        db.session.rollback()
        abort(500, description='An error occurred while unassigning the user')
```

---

Why This Happens:
When you query a Table object using db.session.query(product_assignee_table), the result is a raw SQLAlchemy row (Row object) rather than an ORM-mapped object. Since db.session.delete() expects an ORM-mapped object, it raises this error.


Fix:
To resolve this issue, you need to explicitly delete rows using the delete() method from SQLAlchemy's query API instead of db.session.delete().


```python
def delete_assignee(user_id, product_id):
    try:
        assignee = User.query.filter_by(id=user_id).first()
        product = Product.query.filter_by(id=product_id).first()

        if not assignee:
            abort(404, description='Assignee not found!')

        if not product:
            abort(404, description='Product not found!')

        if assignee not in product.assignees:
            abort(404, description='Assignee not found in product!')

        # Debugging: Print the schema of product_assignee_table
        print(product_assignee_table.columns.keys())

        # Delete the specific row(s) from the association table
        delete_query = product_assignee_table.delete().where(
            (product_assignee_table.c.product_id == product.id) &
            (product_assignee_table.c.assignee_id == assignee.id)
        )
        db.session.execute(delete_query)
        db.session.commit()

        return {'success': True}

    except Exception as e:
        print(e)
        db.session.rollback()
        abort(500, description='An error occurred while unassigning the user')
```


**Explanation of Changes:**

1. Use delete() for Raw Table Rows: 
   * Instead of querying and deleting rows using db.session.delete(), use the delete() method directly on the product_assignee_table.
   
   * The delete() method generates a DELETE SQL statement that removes rows matching the specified condition.

2. Use product_assignee_table.c for Column Access:
   * When working with a Table object, use the .c attribute to access its columns (e.g., product_assignee_table.c.product_id).

3. Execute the Query:
   * Use db.session.execute(delete_query) to execute the DELETE statement.

4. Commit the Transaction:
   * Call db.session.commit() to persist the changes to the database.

---

Debugging Tips:

* Add a debug print statement to log the generated DELETE query:
   
   ``` print(delete_query) ```

* Verify that the product_id and assignee_id values being passed to the query are correct.

  Example Output:
  
  If the DELETE request is successful, the function will return:
  ```json
  {
    "success": true
  }
  ```

If the assignee or product is not found, or if the association does not exist, the function will return an appropriate 404 error.

---