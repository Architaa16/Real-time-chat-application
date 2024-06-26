# Flask Chat Application

This is a simple chat application built with Flask, Flask-SocketIO, and SQLite. It allows multiple users to chat in real-time and stores messages in a SQLite database.

## Requirements

- Python 3.x
- Flask
- Flask-SocketIO
- Eventlet
- SQLite3
- Nest-Asyncio

## Installation

1. Ensure you have Python 3 installed. If not, you can download it from [here](https://www.python.org/downloads/).
2. Install the necessary libraries using pip:
    ```sh
    pip install flask flask-socketio eventlet sqlite3 nest-asyncio
    ```

## Usage

1. Copy the following code into a file named `chat_app.py`.

    ```python
    from flask import Flask, render_template, request
    from flask_socketio import SocketIO, send
    import sqlite3
    from threading import Thread
    import nest_asyncio

    nest_asyncio.apply()
    app = Flask(__name__)
    app.config['SECRET_KEY'] = 'secret!'
    socketio = SocketIO(app)

    def init_db():
        conn = sqlite3.connect('chat.db')
        c = conn.cursor()
        c.execute('''
        CREATE TABLE IF NOT EXISTS messages (
            id INTEGER PRIMARY KEY,
            username TEXT NOT NULL,
            content TEXT NOT NULL,
            timestamp DATETIME DEFAULT CURRENT_TIMESTAMP
        )
        ''')
        conn.commit()
        conn.close()

    init_db()

    @app.route('/')
    def index():
        return '''
        <!doctype html>
        <html>
        <head>
            <title>Chat Application</title>
            <script src="https://cdnjs.cloudflare.com/ajax/libs/socket.io/3.1.3/socket.io.js"></script>
            <script type="text/javascript">
                document.addEventListener("DOMContentLoaded", function() {
                    var socket = io();
                    socket.on('message', function(msg) {
                        var messages = document.getElementById('messages');
                        var message = document.createElement('div');
                        message.innerText = msg.username + ': ' + msg.content;
                        messages.appendChild(message);
                    });

                    document.getElementById('sendBtn').onclick = function() {
                        var username = document.getElementById('username').value;
                        var content = document.getElementById('message').value;
                        socket.send({username: username, content: content});
                    };
                });
            </script>
        </head>
        <body>
            <h1>Chat Room</h1>
            <div id="messages"></div>
            <input type="text" id="username" placeholder="Username">
            <input type="text" id="message" placeholder="Message">
            <button id="sendBtn">Send</button>
        </body>
        </html>
        '''

    @socketio.on('message')
    def handle_message(msg):
        username = msg['username']
        content = msg['content']

        conn = sqlite3.connect('chat.db')
        c = conn.cursor()
        c.execute("INSERT INTO messages (username, content) VALUES (?, ?)", (username, content))
        conn.commit()
        conn.close()

        send(msg, broadcast=True)

    def run_app():
        socketio.run(app, host='0.0.0.0', port=5000)

    if __name__ == '__main__':
        thread = Thread(target=run_app)
        thread.start()
    ```

2. Run the script using Python:
    ```sh
    python chat_app.py
    ```

## Features

- **Real-time Chat**: Multiple users can chat in real-time.
- **Message Storage**: Messages are stored in a SQLite database.
- **Simple Web Interface**: A basic web interface to send and receive messages.

## Notes

- Ensure no other application is using port 5000 to avoid the `OSError: [WinError 10048]` error.
- The server runs on `0.0.0.0` and port `5000`. Adjust these settings as needed.


