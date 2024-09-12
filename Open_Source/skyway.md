I cant record video in my laptop, as it only records game play and cant record windows, but this code can book flight as per my requirement.
Here are the detailed step screenshot at the end of the source code.



import tkinter as tk
from tkinter import messagebox
import mysql.connector
from datetime import datetime

# Database connection
conn = mysql.connector.connect(
    host="localhost",
    user="root",
    password="*********",  
    database="Skyways"
)
cursor = conn.cursor()

# Function to show all available flights
def show_flights():
    cursor.execute("SELECT * FROM flights")
    flights = cursor.fetchall()

    flight_list.delete(0, tk.END)  # Clear listbox
    if flights:
        for flight in flights:
            flight_list.insert(tk.END, f"Flight {flight[0]} | {flight[1]} to {flight[2]} | Price: {flight[3]} | Departs at: {flight[4]}")
    else:
        flight_list.insert(tk.END, "No flights available.")

# Function to book a new flight
def book_flight():
    user_name = entry_user.get()
    start = entry_start.get()
    destination = entry_destination.get()
    departure_time = entry_departure_time.get()
    flight_number = entry_flight_number.get()
    price = entry_price.get()

    if flight_number and start and destination and price and departure_time:
        
        cursor.execute("INSERT INTO flights (flight_number, start, destination, price, departure_time) VALUES (%s, %s, %s, %s, %s)", 
                       (flight_number, start, destination, price, departure_time))
        conn.commit()

        # Book the flight
        cursor.execute("INSERT INTO booked_flights (flight_number, user_name) VALUES (%s, %s)", (flight_number, user_name))
        conn.commit()

        messagebox.showinfo("Success", f"Flight booked: {flight_number} from {start} to {destination} at {departure_time}")
    else:
        messagebox.showwarning("Error", "Please fill all flight details.")

# Function to display booked flight
def show_booked_flight():
    user_name = entry_user.get()

    cursor.execute("""
        SELECT b.id, f.flight_number, f.start, f.destination, f.departure_time 
        FROM booked_flights b 
        JOIN flights f ON b.flight_number = f.flight_number 
        WHERE b.user_name = %s
    """, (user_name,))
    bookings = cursor.fetchall()

    booked_flight_list.delete(0, tk.END)  # Clear listbox
    if bookings:
        for booking in bookings:
            booked_flight_list.insert(tk.END, f"Booking ID: {booking[0]} | Flight {booking[1]} | {booking[2]} to {booking[3]} | Departs at: {booking[4]}")
    else:
        booked_flight_list.insert(tk.END, "No booked flights for this user.")

# Function to cancel a booked flight
def cancel_flight():
    booking_id = entry_booking_id.get()

    cursor.execute("DELETE FROM booked_flights WHERE id = %s", (booking_id,))
    conn.commit()
    messagebox.showinfo("Success", f"Booking ID {booking_id} canceled!")
    show_booked_flight()  # Refresh the bookings list

# GUI setup
root = tk.Tk()
root.title("Skyways Flight Management System")
root.geometry("600x500")  # Adjusted for larger window

# Show available flights UI
btn_show_flights = tk.Button(root, text="Show Available Flights", command=show_flights)
btn_show_flights.grid(row=0, column=0, columnspan=2)

flight_list = tk.Listbox(root, width=80)
flight_list.grid(row=1, column=0, columnspan=2)

# Book Flight UI
tk.Label(root, text="User Name:").grid(row=2, column=0)
entry_user = tk.Entry(root)
entry_user.grid(row=2, column=1)

tk.Label(root, text="Start:").grid(row=3, column=0)
entry_start = tk.Entry(root)
entry_start.grid(row=3, column=1)

tk.Label(root, text="Destination:").grid(row=4, column=0)
entry_destination = tk.Entry(root)
entry_destination.grid(row=4, column=1)

tk.Label(root, text="Flight Number:").grid(row=5, column=0)
entry_flight_number = tk.Entry(root)
entry_flight_number.grid(row=5, column=1)

tk.Label(root, text="Price:").grid(row=6, column=0)
entry_price = tk.Entry(root)
entry_price.grid(row=6, column=1)

tk.Label(root, text="Departure Time (YYYY-MM-DD HH:MM:SS):").grid(row=7, column=0)
entry_departure_time = tk.Entry(root)
entry_departure_time.grid(row=7, column=1)

btn_book = tk.Button(root, text="Book Flight", command=book_flight)
btn_book.grid(row=8, column=0, columnspan=2)

# Show Booked Flights UI
btn_show_booked = tk.Button(root, text="Show My Booked Flight", command=show_booked_flight)
btn_show_booked.grid(row=9, column=0, columnspan=2)

booked_flight_list = tk.Listbox(root, width=80)
booked_flight_list.grid(row=10, column=0, columnspan=2)

# Cancel Flight UI
tk.Label(root, text="Booking ID to Cancel:").grid(row=11, column=0)
entry_booking_id = tk.Entry(root)
entry_booking_id.grid(row=11, column=1)

btn_cancel = tk.Button(root, text="Cancel Flight", command=cancel_flight)
btn_cancel.grid(row=12, column=0, columnspan=2)

root.mainloop()

# Close the database connection
conn.close()



    
