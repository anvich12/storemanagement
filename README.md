# storemanagement
import mysql.connector as mysql
from prettytable import PrettyTable
# Create a database connection
mydb = mysql.connect(host="localhost", user="root", password="root")
mycursor = mydb.cursor()
# Create database if it doesn't exist
mycursor.execute("CREATE DATABASE IF NOT EXISTS store")
mydb.commit()
# Use the created database
mycursor.execute("USE store")
# Create table item if it doesn't exist
s = """CREATE TABLE IF NOT EXISTS item (itemno INT(20) PRIMARY KEY,
        name CHAR(20),brand CHAR(20), price INT(10))"""
mycursor.execute(s)
mydb.commit()
#create table customer if it doesn't exist
s2= """CREATE TABLE IF NOT EXISTS customer(custid int(20)
            ,custname char(20),custadd char(20),
            custphone bigint)"""
mycursor.execute(s2)
mydb.commit()
#create table bill if it doesn't exist
s3="""CREATE TABLE IF NOT EXISTS bill(billno int(20),
      itemno int(20),billdate date,custno char(20),
      noofitems int(20),
     billamt float(20),
      foreign key (itemno) references item (itemno))"""
mycursor.execute(s3)
mydb.commit()

# Menu-driven program for  operations
def menu(mydb, mycursor):
    while True:
        print("++++++++++++++++++++~~~~~~ Welcome to Store ~~~~~~+++++++++++++++++++++++\n\n")
        print("Enter the following Choice for Item record:\n")
        print("1. Add item record")
        print("2. Update item record")
        print("3. Delete item record")
        print("4. Show item record\n")
        print("Enter the following choice for Customer Record:\n")
        print("5. Add customer record")
        print("6. Delete customer record")

        print("7. Modify Customer record\n")
        print("8. Generate bill")
        print("9. Close program")
        try:
            choice = int(input("Enter your choice: "))
        except ValueError as e:
            choice = 0
        if choice == 1:
            adddata(mydb, mycursor)
        elif choice == 2:
            updatedata(mydb, mycursor)
        elif choice == 3:
            deletedata(mydb, mycursor)
        elif choice == 4:
            displaydata(mydb, mycursor)
        elif choice == 5:
            adddatacust(mydb, mycursor)
        elif choice == 6:
            deletedatacust(mydb, mycursor)
        elif choice == 7:
            modifydata(mydb, mycursor)
        elif choice == 8:
            generate(mydb, mycursor)

        elif choice == 9:
            print("Program closed")
            break
        else:
            print("Invalid choice, try again!")

# Define the functions for operations

def adddata(mydb, mycursor):
    itemno = int(input("Enter item number: "))
    name = input("Enter item name: ")
    brand = input("Enter item brand: ")
    price = int(input("Enter item price: "))
    sql = "INSERT INTO item (itemno, name, brand, price) VALUES (%s, %s, %s, %s)"
    val = (itemno, name, brand, price)
    mycursor.execute(sql, val)
    mydb.commit()
    print("Record inserted successfully!")
def updatedata(mydb, mycursor):
    itemno = int(input("Enter item number to update: "))
    name = input("Enter new item name: ")
    brand = input("Enter new item brand: ")
    price = int(input("Enter new item price: "))

    sql = "UPDATE item SET name=%s, brand=%s, price=%s WHERE itemno=%s"
    val = (name, brand, price, itemno)
    mycursor.execute(sql, val)
    mydb.commit()
    print("Record updated successfully!")
def deletedata(mydb, mycursor):
    itemno = int(input("Enter item number to delete: "))
    sql = "DELETE FROM item WHERE itemno=%s"
    val = (itemno,)
    mycursor.execute(sql, val)
    mydb.commit()
    print("Record deleted successfully!")
def displaydata(mydb, mycursor):
    mycursor.execute("SELECT * FROM item")
    result=mycursor.fetchall()
    for x in result:
      print(x)
def adddatacust(mydb, mycursor):
    custid = int(input("Enter customer Id: "))
    custname = input("Enter customer name:")
    custadd= input("Enter customer address:")
    custphone= int(input("Enter customer phone no:")
    ms= "INSERT INTO customer(custid,custname,custadd,custphone) VALUES (%s, %s, %s, %s)"
    lav= (custid, custname, custadd,custphone)
    mycursor.execute(ms, lav)
    mydb.commit()
    print(" Record inserted successfully!")
def deletedatacust(mydb, mycursor):
    custid= int(input("Enter customer ID to delete:"))
    sql= "DELETE FROM customer WHERE custid=%s"
    val = (custid,)
    mycursor.execute(sql, val)
    mydb.commit()
    print("Record deleted succesfully!")
def modifydata(mydb, mycursor):
    custid = int(input("Enter customer Id to be updated : "))
    custname = input("Enter new customer name:")
    custadd= input("Enter new customer address:")
    custphone= int(input("Enter new customer phone no:"))
    sql="UPDATE customer SET custname=%s, custadd=%s,custphone=%s WHERE custid=%s"
    val=(custid,custname,custadd,custphone)
    mycursor.execute(sql,val)
    mydb.commit()

    print("Record updated successfully!")
def generate(mydb,mycursor):
    while True:
        billno = int(input("Enter Bill number: "))
        billdate = input("Enter date (YYYY-MM-DD): ")
        mycursor.execute('Select billno from bill where billno=%s', (billno, ))
        f= mycursor.fetchall()
        if f==[]:
            break
        else:
            print('Bill number already exists.')
    while True:
        custno = int(input("Enter customer id: "))
        mycursor.execute('Select custname from customer where custid=%s', (custno, ))
        custname= mycursor.fetchone()
        if custname is None:
            print("Customer doesn't exist")
        else:
            break

    n= int(input('Number of items: '))
    i=0
    items=[]
    total= 0
    while i<n:
        itemno = int(input("Enter item number: "))
        # Fetch the price of the item based on itemno
        mycursor.execute("SELECT name, price FROM item WHERE itemno = %s", (itemno,))
        result = mycursor.fetchone()
        if result is None:
            print("Item not found!")
        else:
            name, price = result[0],result[1]
            noofitems = int(input("Enter Quantity of items: "))
        # Calculate the total bill amount
            billamt = price * noofitems
            total+=billamt
        # Insert the bill record into the bill table
            items.append((itemno, name, price, noofitems, billamt))
            sql = """INSERT INTO bill (billno, itemno, billdate, noofitems, custno,billamt)
             VALUES (%s, %s, %s, %s, %s,%s)"""
            val = (billno, itemno, billdate, noofitems, custno,billamt)
            mycursor.execute(sql, val)
            mydb.commit()
            i+=1

    print("Bill generated successfully!")
    print('')
    print("Bill Number:",billno,"Customer Name:",custname[0],"Date: ",billdate, "Quantity: ", n)
    print('')
    table= PrettyTable()
    table.field_names= ['Item Number', 'Item Name', 'Price', 'Quantity', 'Tota Price']
    for item in items:
        table.add_row([item[0],item[1], 'INR '+str(item[2]), item[3], 'INR '+str(item[4])])
    print(table)
    print('Grand Total: INR ', total)
    print('')
# Run the menu
menu(mydb, mycursor)
# Close the database connection
mydb.close()
