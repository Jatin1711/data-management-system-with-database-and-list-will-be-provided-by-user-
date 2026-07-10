# data-management-system-with-database-and-list-will-be-provided-by-user-
#it required employ id, name, salary, department, it ask by client not by user
import sqlite3

DB = "employee_database.db"

def connect():
    return sqlite3.connect(DB)

def setup():
    with connect() as c:
        c.execute("""CREATE TABLE IF NOT EXISTS employee(
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT,
            department TEXT,
            salary REAL CHECK(salary >= 0))""")  # DB-level constraint

def add(name, dept, sal):
    if sal < 0:
        print(" Salary cannot be negative!")
        return
    with connect() as c:
        c.execute("INSERT INTO employee(name,department,salary) VALUES(?,?,?)",
                  (name, dept, sal))

def update(emp_id, name=None, dept=None, sal=None):
    if emp_id <= 0:
        print(" ID must be positive!")
        return
    if sal is not None and sal < 0:
        print(" Salary cannot be negative!")
        return

    fields, vals = [], []
    if name:
        fields.append("name=?")
        vals.append(name)
    if dept:
        fields.append("department=?")
        vals.append(dept)
    if sal is not None:
        fields.append("salary=?")
        vals.append(sal)

    if fields:
        vals.append(emp_id)
        with connect() as c:
            c.execute(f"UPDATE employee SET {','.join(fields)} WHERE id=?", vals)

def delete(emp_id):
    if emp_id <= 0:
        print(" ID must be positive!")
        return
    with connect() as c:
        c.execute("DELETE FROM employee WHERE id=?", (emp_id,))

def search(term):
    with connect() as c:
        cur = c.cursor()
        if term.isdigit():
            emp_id = int(term)
            if emp_id <= 0:
                print(" ID must be positive!")
                return
            cur.execute("SELECT * FROM employee WHERE id=? OR name LIKE ?",
                        (emp_id, f"%{term}%"))
        else:
            cur.execute("SELECT * FROM employee WHERE name LIKE ?", (f"%{term}%",))
        print(cur.fetchall())

def list_all():
    with connect() as c:
        for row in c.execute("SELECT * FROM employee"):
            print(row)

def menu():
    setup()
    while True:
        choice = input("\n1.Add 2.Update 3.Delete 4.Search 5.List 6.Exit: ")
        if choice == '1':
            name = input("Name:")
            dept = input("Dept:")
            sal = float(input("Salary:"))
            add(name, dept, sal)

        elif choice == '2':
            emp_id = int(input("ID:"))
            name = input("Name:") or None
            dept = input("Dept:") or None
            sal_in = input("Salary:")
            sal = float(sal_in) if sal_in else None
            update(emp_id, name, dept, sal)

        elif choice == '3':
            emp_id = int(input("ID:"))
            delete(emp_id)

        elif choice == '4':
            search(input("Search term:"))

        elif choice == '5':
            list_all()

        elif choice == '6':
            break

if __name__ == "__main__":
    menu()
