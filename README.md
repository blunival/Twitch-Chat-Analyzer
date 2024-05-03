**TWITCH CHAT ANALYZER**

*@bluni_val*

![image](https://github.com/blunival/Twitch-Chat-Analyzer/assets/168806404/6dffdb74-9ef1-497c-88be-cb43e535f04a)

**Functionality**

- Define the top __ users of a certain emote (case-sensitive obv) and a list will be returned

- Find the 'rank' of a certain username

- There's some basic logic that handles ties, and ties should be sorted alphabetically for your convenience

**Known Bugs**

- Alt-tabbing sends the program to purgatory (minmizes, can't re-open even through task manager)

**Other Stuff of Note**

- I've included some chat log files to mess around with in the github repo (instructions for how to download chat logs from any streamer can be found by pressing (?) in the app - you will have to use [TwitchDownloaderWPF]([GitHub - lay295/TwitchDownloader: Twitch VOD/Clip Downloader - Chat Download/Render/Replay](https://github.com/lay295/TwitchDownloader))).

- Windows Defender and other antivirus software will probably **be very sus** of this program lol. 

- If there's a free way of obtaining a code signing certificate so antivirus programs don't do that I didn't find it lmaoo (idk what i'm doing ngl)

- I've only taken one bioinformatics class 2 years ago and that's it, and that's all my coding experience. The initial code was borked af, and a lot of stuff didn't work like emotes with colons [as a D: and :) user this was very troubling]. Soooo most of the final product was done with the assistance of **Microsoft Copilot**. 

```python
import os
import re
from collections import Counter
import tkinter as tk
from tkinter import Label, Entry, Button, filedialog, Scrollbar, Listbox, ttk, messagebox

# Function to count emotes and find user's rank
def count_emotes_and_find_rank(chat_log_directory, emote_to_find, top, progress_var, root, username):
    user_emote_counts = Counter()
    user_rank = None
    user_count = None

    # Regular expression for the specified emote
    if ':' in emote_to_find:
        emote_pattern = re.compile(re.escape(emote_to_find))
    else:
        emote_pattern = re.compile(r'\b' + re.escape(emote_to_find) + r'\b')

    time_pattern = re.compile(r'\[(\d{1,2}:\d{2}:\d{2})\]')
    username_pattern = re.compile(r'^[a-zA-Z0-9_]{4,25}$')

    files = [f for f in os.listdir(chat_log_directory) if f.endswith('.txt')]
    total_files = len(files)

    # Check if no text files are found
    if total_files == 0:
        messagebox.showerror("Error", "No text files found. Ensure you input the correct directory and added chat logs. For instructions on how to obtain the properly formatted chat logs press the (?)")
        return [], None, None

    for index, filename in enumerate(files, start=1):
        progress_var.set((index / total_files) * 100)
        root.update_idletasks()  # Update the progress bar
        file_path = os.path.join(chat_log_directory, filename)
        with open(file_path, 'r', encoding='utf-8') as file:
            for line in file:
                line = time_pattern.sub('', line).strip()
                if ': ' in line:
                    parts = line.split(': ', 1)
                    if len(parts) == 2:
                        user, message = parts
                        if username_pattern.match(user):
                            emote_count = len(emote_pattern.findall(message))
                            user_emote_counts[user] += emote_count
                            if user == username:
                                user_count = user_emote_counts[user]

    # Sort the user emote counts in descending order
    sorted_emote_counts = sorted(user_emote_counts.items(), key=lambda item: item[1], reverse=True)

    # Logic to handle ties in the ranking
    top_users = []
    last_count = None
    actual_rank = 0
    rank_counter = 1
    for user, count in sorted_emote_counts:
        if count != last_count:
            actual_rank = rank_counter
            last_count = count
        if rank_counter <= top:
            top_users.append((actual_rank, user, count))
        rank_counter += 1

    # Find the rank of the specified username
    if username:
        user_rank = None
        user_count = user_emote_counts[username]
        for rank, user, count in top_users:
            if user == username:
                user_rank = rank
                break
        if not user_rank and user_count:
            # Find the rank for the user if they aren't in the top list
            user_rank = sum(1 for _, count in sorted_emote_counts if count > user_count) + 1

    return top_users, user_rank, user_count

# Function to handle the start of the counting process
def start_counting(emote_entry, top_entry, directory_entry, results_listbox, progress_var, root, user_entry, user_result_label):
    emote_to_find = emote_entry.get()
    top_str = top_entry.get()
    chat_log_directory = directory_entry.get()
    username = user_entry.get().strip()

    # Clear previous results
    results_listbox.delete(0, tk.END)
    user_result_label.config(text="")

    # Error check for emote input
    if not emote_to_find:
        messagebox.showerror("Error", "Please specify an emote/word to find.")
        return

    # Error check for top users input
    if not top_str:
        messagebox.showerror("Error", "Please specify the number of top users to display.")
        return
    try:
        top = int(top_str)
        if top < 1:
            raise ValueError
    except ValueError:
        messagebox.showerror("Error", "The number of top users must be an integer greater than or equal to 1.")
        return

    # Error check for directory input
    if not chat_log_directory:
        messagebox.showerror("Error", "Please specify the chat logs directory.")
        return

    # Start the counting process if all inputs are valid
    top_users, user_rank, user_count = count_emotes_and_find_rank(chat_log_directory, emote_to_find, top, progress_var, root, username)

    # Display the results in the listbox
    for rank, user, count in top_users:
        results_listbox.insert(tk.END, f"{rank}. {user} - {count} times")

    # Display user's rank
    if user_rank is not None:
        user_result_label.config(text=f"Your rank: {user_rank}, with {user_count} uses of the emote.")
    elif username:
        user_result_label.config(text="You did not use the emote/word.")

# Function to browse for the chat log directory
def browse_directory(directory_entry):
    dirname = filedialog.askdirectory()
    if not dirname:
        messagebox.showerror("Error", "Please select a directory.")
        return
    directory_entry.delete(0, tk.END)
    directory_entry.insert(0, dirname)

# Function to show help message in a copyable text window
def show_help():
    help_window = tk.Toplevel()
    help_window.title("Help")
    help_text = tk.Text(help_window, wrap="word", height=15, width=50)
    help_text.pack(padx=10, pady=10)
    help_message = """1) Download TwitchDownloaderWPF from https://github.com/lay295/TwitchDownloader/blob/master/TwitchDownloaderWPF/README.md
2) Launch TwitchDownloaderWPF and go to "Task Queue".
3) Under "Mass Downloads" click Search VODs.
4) Set the desired channel and select the desired VODs and press "Add To Queue".
5) Select "Download Chat" and select "TXT" as the download format, then press "Add to Queue".
6) Select the directory where you saved the chat logs and you're good to go :)"""
    help_text.insert("1.0", help_message)
    help_text["state"] = "disabled"  # Make the text read-only

# Main GUI application
def main_app():
    try:
        root = tk.Tk()
        root.title("Twitch Chat Analyzer (@bluni_val)")
        root.geometry("400x400")  # Set the window size

        # Remove the default favicon
        root.wm_attributes('-toolwindow', 'True')

        def on_minimize():
            root.iconify()

        def on_restore():
            root.deiconify()

        # Bind the minimize event to the on_minimize function
        root.bind('<Unmap>', lambda event: on_minimize())
        root.bind('<Map>', lambda event: on_restore())

        # Ensure the application is properly closed when the window manager's close button is clicked
        root.protocol("WM_DELETE_WINDOW", root.destroy)

        # Use a frame to contain all widgets
        main_frame = tk.Frame(root, padx=10, pady=10)
        main_frame.pack(expand=True)

        # Create input fields for emote, top number, directory, and username
        Label(main_frame, text="Emote/word:").grid(row=0, column=0, sticky='w')
        emote_entry = Entry(main_frame, width=20)
        emote_entry.grid(row=0, column=1, pady=5)

        Label(main_frame, text="Show top __ users:").grid(row=1, column=0, sticky='w')
        top_entry = Entry(main_frame, width=20)
        top_entry.grid(row=1, column=1, pady=5)

        Label(main_frame, text="Chat Logs Directory:").grid(row=2, column=0, sticky='w')
        directory_entry = Entry(main_frame, width=20)
        directory_entry.grid(row=2, column=1, pady=5)
        browse_button = Button(main_frame, text="Browse", command=lambda: browse_directory(directory_entry))
        browse_button.grid(row=2, column=2, padx=5)

        # Help button
        help_button = Button(main_frame, text="?", command=show_help, height=1, width=2)
        help_button.grid(row=2, column=3, padx=5)

        Label(main_frame, text="Your Username (Optional):").grid(row=3, column=0, sticky='w')
        user_entry = Entry(main_frame, width=20)
        user_entry.grid(row=3, column=1, pady=5)

        # Progress bar
        progress_var = tk.DoubleVar()
        progress_bar = ttk.Progressbar(main_frame, variable=progress_var, maximum=100)
        progress_bar.grid(row=4, column=0, columnspan=3, sticky='we', pady=5)

        # Results listbox with a scrollbar
        results_frame = tk.Frame(main_frame)
        results_frame.grid(row=6, column=0, columnspan=3, sticky='we')
        scrollbar = Scrollbar(results_frame)
        scrollbar.pack(side=tk.RIGHT, fill=tk.Y)
        results_listbox = Listbox(results_frame, yscrollcommand=scrollbar.set, width=50, height=10)
        results_listbox.pack(side=tk.LEFT, fill=tk.BOTH, expand=True)
        scrollbar.config(command=results_listbox.yview)

        # User result label
        user_result_label = Label(main_frame, text="", anchor='w')
        user_result_label.grid(row=7, column=0, columnspan=3, sticky='we')

        # Button to start the counting process
        start_button = Button(main_frame, text="Start", command=lambda: start_counting(emote_entry, top_entry, directory_entry, results_listbox, progress_var, root, user_entry, user_result_label))
        start_button.grid(row=5, columnspan=3, pady=10)

        root.mainloop()
        
    except Exception as e:
        messagebox.showerror("Error", f"An unexpected error occurred: {e}")
        raise  # Re-raise the exception for debugging purposes

# Run the application
if __name__ == "__main__":
    main_app()

```
