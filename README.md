# ATM_innovation-
import mysql.connector
from mysql.connector import Error

class ATM:
    def __init__(self, username, password):
        self.username = username
        self.password = password
        self.connection = self.connect_to_db()

    def connect_to_db(self):
        try:
            connection = mysql.connector.connect(
                host='localhost',  # MySQL host
                database='atm_system',  # Database name
                user='root',  # MySQL username
                password='your_password'  # MySQL password
            )
            if connection.is_connected():
                print("Connected to the MySQL database")
                return connection
        except Error as e:
            print(f"Error: {e}")
            return None

    def authenticate_user(self):
        cursor = self.connection.cursor()
        query = "SELECT * FROM accounts WHERE username = %s AND password = %s"
        cursor.execute(query, (self.username, self.password))
        user = cursor.fetchone()
        cursor.close()
        if user:
            print("Authentication successful")
            return user
        else:
            print("Invalid username or password")
            return None

    def check_balance(self):
        cursor = self.connection.cursor()
        query = "SELECT balance FROM accounts WHERE username = %s"
        cursor.execute(query, (self.username,))
        balance = cursor.fetchone()[0]
        cursor.close()
        print(f"Your current balance is: ${balance:.2f}")
        return balance

    def deposit(self, amount):
        if amount <= 0:
            print("Invalid deposit amount!")
            return
        cursor = self.connection.cursor()
        query = "UPDATE accounts SET balance = balance + %s WHERE username = %s"
        cursor.execute(query, (amount, self.username))
        self.connection.commit()
        cursor.close()
        print(f"Deposited ${amount:.2f}.")
        self.check_balance()

    def withdraw(self, amount):
        balance = self.check_balance()
        if amount <= 0:
            print("Invalid withdrawal amount!")
        elif amount > balance:
            print("Insufficient funds!")
        else:
            cursor = self.connection.cursor()
            query = "UPDATE accounts SET balance = balance - %s WHERE username = %s"
            cursor.execute(query, (amount, self.username))
            self.connection.commit()
            cursor.close()
            print(f"Withdrew ${amount:.2f}.")
            self.check_balance()

def atm_menu():
    print("Welcome to the ATM")
    print("1. Check Balance")
    print("2. Deposit Money")
    print("3. Withdraw Money")
    print("4. Exit")

def main():
    username = input("Enter your username: ")
    password = input("Enter your password: ")

    atm = ATM(username, password)
    user = atm.authenticate_user()

    if user:
        while True:
            atm_menu()
            choice = input("Please choose an option (1-4): ")

            if choice == '1':
                atm.check_balance()
            elif choice == '2':
                try:
                    deposit_amount = float(input("Enter the amount to deposit: $"))
                    atm.deposit(deposit_amount)
                except ValueError:
                    print("Invalid input! Please enter a valid number.")
            elif choice == '3':
                try:
                    withdraw_amount = float(input("Enter the amount to withdraw: $"))
                    atm.withdraw(withdraw_amount)
                except ValueError:
                    print("Invalid input! Please enter a valid number.")
            elif choice == '4':
                print("Thank you for using the ATM. Goodbye!")
                break
            else:
                print("Invalid choice. Please try again.")
    else:
        print("Exiting the program...")

if __name__ == "__main__":
    main()
