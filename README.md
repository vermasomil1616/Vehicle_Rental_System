# Vehicle_Rental_System

import tkinter as tk
from tkinter import messagebox
import mysql.connector

class Vehicle:
    def __init__(self, reg_no, make, model, year, available=True):
        self.reg_no = reg_no
        self.make = make
        self.model = model
        self.year = year
        self.available = available

    def __str__(self):
        return f"Reg No: {self.reg_no}, Make: {self.make}, Model: {self.model}, Year: {self.year}, Available: {self.available}"

class VehicleRentalSystem:
    def __init__(self):
        self.db_connection = mysql.connector.connect(
            host="localhost",
            user="root",                 # Replace with your MySQL username
            password="12345@",           # Replace with your MySQL password
            database="vehicle_rental"
        )
        self.db_connection.autocommit = True
        self.cursor = self.db_connection.cursor()

    def load_data(self):
        self.vehicles = []
        query = "SELECT reg_no, make, model, year, available FROM vehicles"
        self.cursor.execute(query)
        result = self.cursor.fetchall()
        for row in result:
            reg_no, make, model, year, available = row
            self.vehicles.append(Vehicle(reg_no, make, model, year, available))

    def rent_vehicle(self, reg_no):
        query = "SELECT available FROM vehicles WHERE reg_no = %s"
        self.cursor.execute(query, (reg_no,))
        result = self.cursor.fetchone()
        if result and result[0]:  # If available is True
            update_query = "UPDATE vehicles SET available = FALSE WHERE reg_no = %s"
            self.cursor.execute(update_query, (reg_no,))
            messagebox.showinfo("Success", f"Vehicle with registration number {reg_no} rented successfully.")
        else:
            messagebox.showerror("Error", "Vehicle not found or not available for rent.")

    def return_vehicle(self, reg_no):
        query = "SELECT available FROM vehicles WHERE reg_no = %s"
        self.cursor.execute(query, (reg_no,))
        result = self.cursor.fetchone()
        if result and not result[0]:  # If available is False
            update_query = "UPDATE vehicles SET available = TRUE WHERE reg_no = %s"
            self.cursor.execute(update_query, (reg_no,))
            messagebox.showinfo("Success", f"Vehicle with registration number {reg_no} returned successfully.")
        else:
            messagebox.showerror("Error", "Vehicle not found or already available.")

    def display_available_vehicles(self):
        available_vehicles = []
        query = "SELECT reg_no, make, model, year FROM vehicles WHERE available = TRUE"
        self.cursor.execute(query)
        result = self.cursor.fetchall()
        return result

    def close_connection(self):
        self.db_connection.close()

class LoginSystem:
    def __init__(self):
        self.root = tk.Tk()
        self.root.title("Vehicle Rental System Login")

        tk.Label(self.root, text="Username:").pack()
        self.username_entry = tk.Entry(self.root)
        self.username_entry.pack()

        tk.Label(self.root, text="Password:").pack()
        self.password_entry = tk.Entry(self.root, show="*")
        self.password_entry.pack()

        tk.Button(self.root, text="Login", command=self.check_credentials).pack()

    def check_credentials(self):
        username = self.username_entry.get()
        password = self.password_entry.get()

        if username == "somil" and password == "@12345":
            self.root.destroy()
            self.run_vehicle_rental_system()
        else:
            messagebox.showerror("Invalid Credentials", "Username or password is incorrect.")

    def run_vehicle_rental_system(self):
        rental_system = VehicleRentalSystem()
        rental_system.load_data()
        self.main_screen(rental_system)

    def main_screen(self, rental_system):
        main_window = tk.Tk()
        main_window.title("Vehicle Rental System")

        tk.Button(main_window, text="Rent a Vehicle", command=lambda: self.rent_vehicle(rental_system)).pack()
        tk.Button(main_window, text="Return a Vehicle", command=lambda: self.return_vehicle(rental_system)).pack()
        tk.Button(main_window, text="Display Available Vehicles", command=lambda: self.display_available_vehicles(rental_system)).pack()
        tk.Button(main_window, text="Exit", command=lambda: self.exit_system(rental_system, main_window)).pack()

        main_window.mainloop()

    def rent_vehicle(self, rental_system):
        def rent():
            reg_no = reg_no_entry.get()
            rental_system.rent_vehicle(reg_no)
            rent_window.destroy()

        rent_window = tk.Toplevel()
        rent_window.title("Rent a Vehicle")

        tk.Label(rent_window, text="Enter Vehicle Registration Number:").pack()
        reg_no_entry = tk.Entry(rent_window)
        reg_no_entry.pack()

        tk.Button(rent_window, text="Rent", command=rent).pack()

    def return_vehicle(self, rental_system):
        def return_veh():
            reg_no = reg_no_entry.get()
            rental_system.return_vehicle(reg_no)
            return_window.destroy()

        return_window = tk.Toplevel()
        return_window.title("Return a Vehicle")

        tk.Label(return_window, text="Enter Vehicle Registration Number:").pack()
        reg_no_entry = tk.Entry(return_window)
        reg_no_entry.pack()

        tk.Button(return_window, text="Return", command=return_veh).pack()

    def display_available_vehicles(self, rental_system):
        available_window = tk.Toplevel()
        available_window.title("Available Vehicles")

        result = rental_system.display_available_vehicles()
        if result:
            for row in result:
                vehicle_info = f"Reg No: {row[0]}, Make: {row[1]}, Model: {row[2]}, Year: {row[3]}"
                tk.Label(available_window, text=vehicle_info).pack()
        else:
            tk.Label(available_window, text="No vehicles available for rent.").pack()

    def exit_system(self, rental_system, main_window):
        rental_system.close_connection()
        main_window.quit()

    def run(self):
        self.root.mainloop()

if __name__ == "__main__":
    login_system = LoginSystem()
    login_system.run()
