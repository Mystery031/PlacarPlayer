import mysql.connector
import tkinter as tk
from tkinter import messagebox

def main():
    # Initialize the database connection
    db_config = {
        "host": "localhost",
        "user": "your_username",
        "password": "your_password",
        "database": "your_database"
    }
    connection = mysql.connector.connect(**db_config)

    if connection.is_connected():
        # Create the main window
        main_window = tk.Tk()
        main_window.title("Player & Match Management")

        # Add the UI components
        tk.Label(main_window, text="Player Name:").pack()
        player_name_entry = tk.Entry(main_window)
        player_name_entry.pack()

        tk.Button(main_window, text="Add Player", command=lambda: insert_player(connection, player_name_entry.get())).pack()

        tk.Label(main_window, text="Player Team ID:").pack()
        player_team_id_entry = tk.Entry(main_window)
        player_team_id_entry.pack()

        tk.Label(main_window, text="Match Results:").pack()
        team1_score_entry = tk.Entry(main_window)
        team1_score_entry.pack()
        team2_score_entry = tk.Entry(main_window)
        team2_score_entry.pack()

        tk.Button(main_window, text="Update Ratings", command=lambda: update_ratings(connection, team1_score_entry.get(), team2_score_entry.get(), player_team_id_entry.get())).pack()

        # Start the main loop
        main_window.mainloop()

        # Close the database connection
        connection.close()
    else:
        print("Failed to connect to the database.")

def insert_player(conn, name):
    cursor = conn.cursor()
    try:
        cursor.execute("INSERT INTO Player (name, team_id) VALUES (%s, %s)", (name, 1))
        conn.commit()
        messagebox.showinfo("Player Added", f"Player '{name}' was added to the database.")
    except mysql.connector.Error as error:
        messagebox.showerror("Error", f"Error adding player: {error}")

def update_ratings(conn, team1_score, team2_score, team_id):
    if team1_score.strip() == "" or team2_score.strip() == "" or team_id.strip() == "":
        messagebox.showerror("Error", "Please fill in all fields.")
        return

    try:
        team1_score = int(team1_score)
        team2_score = int(team2_score)
        team_id = int(team_id)
    except ValueError:
        messagebox.showerror("Error", "Scores and Team ID must be integers.")
        return

    cursor = conn.cursor()
    try:
        if team1_score > team2_score:
            cursor.execute("UPDATE Team SET rating = rating + 3 WHERE team_id = %s", (team_id,))
        elif team1_score < team2_score:
            cursor.execute("UPDATE Team SET rating = rating + 3 WHERE team_id = %s", (3 - team_id,))
        else:
            cursor.execute("UPDATE Team SET rating = rating + 1 WHERE team_id IN (%s, %s)", (team_id, 3 - team_id))
        conn.commit()
        messagebox.showinfo("Ratings Updated", "Team ratings were updated successfully.")
    except mysql.connector.Error as error:
        messagebox.showerror("Error", f"Error updating ratings: {error}")

if __name__ == "__main__":
    main()
