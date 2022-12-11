### Creating a Simple Hunt the Wumpus Game with Python, Flask, and SocketIO

#### Introduction

One of the first computer games is Hunt the Wumpus. It is a simple adventure game that was developed in 1973. In Hunt the Wumpus, the player is an adventurer entering a dungeon looking for a treasure. The goal of the adventurer is to find the treasure and get out of the dungeon avoiding the pits where he can fall in and the Wumpus: a monster who kills anyone it sees. To do that, the player has to pay attention to the signals: when he is near a pit he can feel a breeze, and when he is near the Wumpus he can smell the Wumpus' stink.

Hunt the Wumpus was studied by computer scientists in the early days of the artificial intelligence field and it exemplifies the use of logic in A.I. In this tutorial, we are going to build a simple graphical web-based Hunt the Wumpus game using Python, Flask, and SocketIO.

#### Pre-requisites

To follow this tutorial, you should know:

- How to create the basic structure of a page using HTML and CSS;
- How to interact with elements of a web page using JavaScript and jQuery;
- The basics of Python programming;
- The basics of Flask.

This tutorial will **not** teach the basics of how to build a web page with HTML, CSS, and Javascript, nor how Flask works. The tutorial will assume you already know this and focus on the development of the Hunt the Wumpus game using these technologies.

That said, if you're a complete beginner on these technologies, you may still be able to follow the tutorial without major difficulties. Whenever you get a doubt, go to the documentation referenced at the end of the article and it's likely you will be fine.

#### The Hunt the Wumpus game: brief history and contributions to AI

Hunt the Wumpus is a text-based adventure game developed in 1973 by Gregory Yob for PC.  In the game, the player enters a dungeon made of several conneced rooms and has to avoid pits where he can fall in, bats that can move him to a random room, and kill the monster Wumpus with an arrow. The player knows there's a pit nearby when he feels a breeze and he knows when there's a Wumpus nearby when he feels a stink.

The "Wumpus World" is a slightly modified version of the Hunt the Wumpus game mentioned in the influential book *Artificial Intelligence: a Modern Approach*. It's described in the book like this

*"The wumpus world is a cave consisting of rooms connected by passageways. Lurking somewhere in the cave is the terrible wumpus, a beast that eats anyone who enters its room. The wumpus can be shot by an agent, but the agent has only one arrow. Some rooms contain bottomless pits that will trap anyone who wanders into these rooms (except for the wumpus, which is too big to fall in). The only mitigating feature of this bleak environment is the possibility of finding a heap of gold."* (Artificial Intelligence: A Modern Approach)

This Wumpus World exemplifies the use of propositional logic in A.I. to build knowledge-based agents. The book shows how we can build an agent able to play Wumpus World "rationally", that is, choosing the safest rooms based on the knowledge it has available.

In this tutorial, we are going to develop a game similar to Wumpus World. Instead of being a text-based game, we are going o build a graphical game, although very simple. And instead of killing the Wumpus, we are going to set the goal to find the treasure. Let's also leave the bad bats behind, and use just the pits and the Wumpus.

#### What is Socket.io and how does it work?

Socket.io is a library that allow us to build apps with bidirectional communication between server and client using a event-based approach. With Socket.io we can *emit* events with messages and *hear* the events and messages. This way we can create sends communication between the server and the client. One side emits events and send a message to the other. The other side receives the message with some data and do some processing with the data. Then it can emit another event sending back the processed data. 

We are going to use Socket.io because we can't let the informations about the dungeon on the client-side. This is information that must be hidden from the player. Only when the player enters a room he will know what there's around. That's what we are going to do:

- We will store the dungeon information on the server

- When the player takes some move on the client-side, we are going to emit an event asking for the information of the new player's position

- The server will hear the client, process the player position, and send another event with the information of the player's surroundings

- The client will hear the new information and show it to the player

#### Step 1 - Project structure and Hello World

The project structure is very simple. It's the same structure as practically any other Flask project. First, create a directory for the project. Inside the project directory, add a file called `wsgi.py` and a directory called `app`. Inside `app/`, add the file `app.py`, and create two other directories: `static` and `templates`. Inside `templates/`, add two files: `base.html` and `index.html`. In `static/`, create two more directories: `scripts` and `styles`. Create the file `main.js` in `scripts/`, and `main.css` in `styles/`. Now it's done! This is the basic structure for a Flask app. Your directory structure should be looking like this:

```
|   wsgi.py
|   
+---app
|   |   app.py
|   |   
|   +---static
|   |   +---scripts
|   |   |       main.js
|   |   |       
|   |   \---styles
|   |           main.css
|   |           
|   +---templates
|   |       base.html
|   |       index.html
```

Create a virtual environment using `python -m venv myenv` and activate it. Install the following packages:

- Flask: `python -m pip install Flask`

- Flask-SocketIO: `python -m pip install Flask-SocketIO`

- Flask-Session: `python -m pip install Flask-Session`

Now let's define the initial visuals of the app inside the `base.html` and `main.css`. Add the meta information of the page, and create the header and the bottom in `base.html`. Let's also add the fonts from Google Fonts, the links to `main.css` and `main.js`, and the CDNs to socket.io and jQuery. Feel free to create the visuals of the header and bottom as you want, but don't forget to add the links and CDNs. Add also the CDN to Feather Icons, so that we can also use nice icons in the interface. 

Between the header and the bottom, you should also have the following code: `{% block content %}{% endblock %}`. This is where we are going to put the page content.

```HTML
<!DOCTYPE html>
<html lang="en">
    <head>
        <title>Hunt the Wumpus</title>

        <meta charset="utf-8"/>
        <meta name="description" content="In Hunt the Wumpus, the player is an adventurer entering a dungeon looking for a treasure. The goal of the adventurer is to find the treasure and get out of the dungeon avoiding the pits where he can fall in and the Wumpus: a monster who kills anyone it sees.">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">

        <!--Fonts from Google Fonts-->
        <!--Eczar-->
        <link rel="preconnect" href="https://fonts.googleapis.com">
        <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
        <link href="https://fonts.googleapis.com/css2?family=Eczar:wght@400;500;600;700;800&display=swap" rel="stylesheet"> 

        <!--CSS-->
        <link rel= "stylesheet" type= "text/css" href= "{{ url_for('static',filename='styles/main.css') }}">

        <!--JavaScript-->
        <!--Socketio-->
        <script src="https://cdn.socket.io/4.4.1/socket.io.min.js" integrity="sha384-fKnu0iswBIqkjxrhQCTZ7qlLHOFEgNkRmK2vaO/LbTZSXdJfAu6ewRBdwHPhBo/H" crossorigin="anonymous"></script>
        <!--jQuery-->
        <script src="https://code.jquery.com/jquery-3.6.0.min.js" integrity="sha256-/xUj+3OJU5yExlq6GSYGSHk7tPXikynS7ogEvDej/m4=" crossorigin="anonymous"></script>
        <!--Feather Icons-->
        <script src="https://cdn.jsdelivr.net/npm/feather-icons/dist/feather.min.js"></script>
        <!--main.js-->
        <script src="{{ url_for('static', filename='scripts/main.js') }}"></script>
    </head>
    <body>

        <div class='header'>
            <div class='title'>
                <a href='#'><h2>Hunt the Wumpus</h2></a>
            </div>
        </div>

        {% block content %}
        {% endblock %}

        <div class='bottom'>
            <p>Created by <em>your name here</em>, 2022</p>
        </div>

    </body>
</html>
```

Add some styles too in `main.css`:

```CSS
html, body {
    margin: 0;
    padding: 0;
    font-family: 'Eczar', 'serif';
    width: 100%;
    height: 100%;
    display: flex;
    flex-direction: column;
    background-color: rgb(240, 220, 180);
    color: rgb(100, 10, 10);
}

a {
    color: rgb(100, 10, 10);
    text-decoration-line: underline;
    text-decoration-style: wavy;
}

h1, h2 {
    line-height: 100%;
}

.header {
    width: 100%;
    display: flex;
    flex-direction: row;
    align-items: center;
    justify-content: center;
}

.bottom {
    margin-top: auto;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
}
```

Let's now extend the `base.html` in `index.html` and add a "Hello World!" to the page.

```HTML
{% extends 'base.html' %}

{% block content %}

<div class='main'>
    <h1>Hello World!</h1>
</div>

{% endblock %}
```

Let's add some style to the main div in `main.css`:

```CSS
.main {
    display: flex;
    flex-direction: column;
    flex-grow: 1;
    align-items: center;
    justify-content: center;
}
```

Now, in `app.py` add:

```python
from flask import Flask, render_template
from flask_socketio import SocketIO, emit
from uuid import uuid4

app = Flask(__name__)
app.config['SECRET_KEY'] = uuid4().hex

socketio = SocketIO(app)

@app.route("/")
def hello_world():
    return render_template('index.html')
```

Finally, add to `wsgi.py`:

```python
from app.app import app, socketio

if __name__ == "__main__":
    socketio.run(app, debug=True)
```

You can now run `python wsgi.py` in your terminal and go to `http://localhost:5000` in your browser. You should see something like this:

![Screenshot of Hello World page](/images/Screenshot_1.png)

#### Step 2 - The content of `index.html`

Now that we created the base template, it's time to create the content of the page. Delete the `<h1>Hello World!</h1>` from `index.html` and let's add a div with information about the game inside `.main`:

```html
<div class='info'>
    <h1>Welcome, Noble Adventurer!</h1>
    <p>In <strong>Hunt the Wumpus</strong>, you are an adventurer entering a dungeon looking for a <em>treasure</em>.</p>
    <p>Your goal is to find the treasure and get out of the dungeon <em>avoiding</em> the <strong>pits where you can fall in</strong> and the </strong>Wumpus</strong>: <em>a monster who kills <strong>anyone</strong> it sees.</em></p>
</div>
```

Let's also add a `form` where the player can select the dimensions of the dungeon:

```html
<div class='setup'>
    <h1>Select the size of the dungeon</h1>
    <form method='POST'>
        <input type="number" id="height" class="size_input" name="height" min="5" max="10" value="5"/>
        <large>X</large> 
        <input type="number" id="width" class="size_input" name="width" min="5" max="10" value="5"/>
        <button class='play' type="submit">Enter the Dungeon</button>
    </form>
</div>
```

Let's stylize the elements we just added in `main.css`:

```css
.info {
    width: 60%;
    text-align: center;
    padding: 30px;
    background-color: rgb(100, 10, 10);
    color:rgb(240, 220, 180);
}

.setup {
    text-align: center;
    padding: 30px;
    display: flex;
    flex-direction: column;
}

.play {
    padding: 10px 15px 10px 15px;
    background-color: rgb(100, 10, 10);
    color: rgb(240, 220, 180);
    border: none;
    font-weight: 600;
    font-family: 'Eczar', 'serif';
    font-size: large;
    cursor: pointer;
    margin: 5px;
}

.size_input {
    padding: 10px 15px 10px 15px;
    border: 1px solid rgb(100, 10, 10);
    color:rgb(100, 10, 10);
    font-family: 'Eczar', 'serif';
    font-size: large;
    width: 43px;
}
```

Refresh your page and now it should be looking like this:

![Index visual ready](/images/Screenshot_2.png)

But the page still doesn't do anything. Now let's make it work!

#### Step 3 - Developing the base of the game

Let's go back to `app.py`. It's where the magic happens. Let's create sessions to store some user data using `flask-session`. 

```python
from flask import session
from flask_session import Session
```

Let's configure the session, making it permanent and setting the session type to `filesystem`. Add this to the beginning of your code:

```python
app.config["SESSION_PERMANENT"] = True
app.config["SESSION_TYPE"] = "filesystem"
Session(app)
```

The first thing we are going to store in the session is the user socket id. We are going to store it when the user first connects to the page using `@socketio.on('connect')`:

```python
@socketio.on('connect')
def connect():
    session['sid'] = request.sid
```

Now let's get the data from the `form` in `index.html`. First, import `request` from `flask`:

```python
from flask import request
```

Now, we need to inform flask that we can receive POST requests from `/` route. Change the `@app.route("/")` to `@app.route("/", methods=['GET', 'POST'])`. Let's also change the function name from `hello_world()` to `index()`. Inside `index()`, we can check if there was a POST request and get the values from the `form`.

```python
@app.route("/", methods=['GET', 'POST'])
def index():
    if request.method == 'POST':
        session['width'] = int(request.form['width'])
        session['height'] = int(request.form['height'])
        return render_template('game.html')
    return render_template('index.html')
```

When we receive a POST request we store the width and height of the dungeon in `session['width']` and `session['height']`. Then, we are rendering the page `game.html`. Note that we didn't create the `game.html` page yet. Go to `templates/` and create it. Let's make it extend from `base.html` too.

```html
{% extends 'base.html' %}

{% block content %}

<div class='main'>
    <!--Nothing in here yet-->
</div>

{% endblock %}
```

Now if you refresh the page in your browser and click on the "Enter the Dungeon" button, the page `game.html` will be rendered. There's nothing in `game.html` yet, so you probably will see just a blank page with only the header and the bottom.

Also, note that we stored the sizes of the dungeon but we didn't do anything with them. Let's use them to create the dungeon now. We are going to store the dungeon as a `height` x `width` matrix -- a list of lists in Python. 

Let's import `random` so that we can generate the random positions of the Wumpus, the Treasure, and the pits.

```python
import random
```

Create a function called `generate_dungeon(w, h)`. The parameter of the function: `w` and `h` are the `width` and `height` of the dungeon. We are going to store the dungeon in a session variable. Let's also get random positions for the Wumpus and the Treasure.

```python
def generate_dungeon(w, h):
    session['dungeon'] = []
    wumpus_x = random.randrange(2, w)
    wumpus_y = random.randrange(2, h)
    gold_x = random.randrange(2, w)
    gold_y = random.randrange(2, h)
    while (gold_x == wumpus_x and gold_y == wumpus_y):
        gold_x = random.randrange(2, w)
        gold_y = random.randrange(2, h)
```

Note that we are getting random coordinates `(x, y)` such that `2 <= x < width` and `2 <= y < height`. This is because the player will start in the position `(0, 0)` of the dungeon and if there's a wumpus or a treasure in the positions `(0, 0)`, `(0, 1)`, `(1, 0)`, `(1, 1)`, it would be very easy to win or lose the game. So let's avoid this by making the Wumpus and the treasure to not appear in those positions. It would also be very tragic if the Wumpus and the treasure appear in the same position. We are avoiding this with the `while` loop that looks for another position for the treasure if it's in the same position as the Wumpus.

Now let's build our matrix line by line:

```python
for i in range(h):
        line = []
        # 0 - free
        # 1 - pit
        # 2 - wumpus
        for j in range(w):
            if (i == 0 or i == 1) and (j == 0 or j == 1):
                # the beginning of the dungeon cannot have pits or wumpus
                line.append(0)
            elif i == wumpus_x and j == wumpus_y:
                # wumpus location
                line.append(2)
            elif i == gold_x and j == gold_y:
                # gold location
                line.append(3)
            elif ((i == wumpus_x - 1 or i == wumpus_x + 1) and (j == wumpus_y)) or ((j == wumpus_y - 1 or j == wumpus_y + 1) and (i == wumpus_x)):
                # nothing around the wumpus
                line.append(0)
            else:
                # random pits distributed
                if random.random() <= 0.15:
                    line.append(1)
                else:
                    line.append(0)
                
        session['dungeon'].append(line)
```

Note that we are also avoiding pits at the beginning of the dungeon, making the positions `(0, 0)`, `(0, 1)` ,`(1, 0)` ,`(1, 1)` free. We are positioning the Wumpus and the treasure, and we are also making the positions around the wumpus free of pits. The pits are positioned in the rest of the map with a probability of 15% of appearing in some position.

Now let's also store the player's initial position as the current position in the session and make `generate_dungeon` call `game.html` with the dungeon already created. Add this to the end of the function:

```python
session['curr_pos'] = [0, 0]
return render_template('game.html', width=w, height=h)
```

It should be looking like this now:

```python
def generate_dungeon(w, h):
    session['dungeon'] = []
    wumpus_x = random.randrange(2, w)
    wumpus_y = random.randrange(2, h)
    gold_x = random.randrange(2, w)
    gold_y = random.randrange(2, h)
    while (gold_x == wumpus_x and gold_y == wumpus_y):
        gold_x = random.randrange(2, w)
        gold_y = random.randrange(2, h)
    for i in range(h):
        line = []
        # 0 - free
        # 1 - pit
        # 2 - wumpus
        for j in range(w):
            if (i == 0 or i == 1) and (j == 0 or j == 1):
                # the beginning of the dungeon cannot have pits or wumpus
                line.append(0)
            elif i == wumpus_x and j == wumpus_y:
                # wumpus location
                line.append(2)
            elif i == gold_x and j == gold_y:
                # gold location
                line.append(3)
            elif ((i == wumpus_x - 1 or i == wumpus_x + 1) and (j == wumpus_y)) or ((j == wumpus_y - 1 or j == wumpus_y + 1) and (i == wumpus_x)):
                # nothing around the wumpus
                line.append(0)
            else:
                # random pits distributed
                if random.random() <= 0.15:
                    line.append(1)
                else:
                    line.append(0)
                
        session['dungeon'].append(line)
    session['curr_pos'] = [0, 0]
    return render_template('game.html', width=w, height=h)
```

In `index()`, delete `return render_template('game.html')` and add `return generate_dungeon(session['width'], session['height'])`. This is how `index()` should be looking like:

```python
@app.route("/", methods=['GET', 'POST'])
def index():
    if request.method == 'POST':
        session['width'] = int(request.form['width'])
        session['height'] = int(request.form['height'])
        return generate_dungeon(session['width'], session['height'])
    return render_template('index.html')
```

Now when the player select the dimensions of the dungeon and click on "Enter the Dungeon", we are creating the dungeon and rendering `game.html`, passing the `width` and `height` of the dungeon as parameters.

Now it's time to create the content of `game.html`. Let's create two divs: `win` and `lose`, with the messages that will appear when the player wins or loses the game. Inside `.main` div add:

```html
<div class="win">
    <h4>Congratulations! You won!</h4>
    <a href='/'><button class='play'>Play again</button></a>
</div>

<div class="lose">
    <h4>Game Over!</h4>
    <a href='/'><button class='play'>Play again</button></a>
</div>
```

Let's also create the dungeon map. Below the `win` and `lose` divs, add:

```html
{% for i in range(height) %}

    <div class="line">
        
    {% for j in range(width) %}

        {% if i == 0 and j == 0 %}

            <div class="room curr_room" x="{{j}}" y="{{i}}"></div>

        {% else %}

            <div class="room" x="{{j}}" y="{{i}}"></div>

        {% endif %}

    {% endfor %}

    </div>

{% endfor %}
```

We are creating a line for each `i` in the range `[0, height)`. Inside each line, we add a `.room`. If the position is `(0, 0)`, we make the `.room` a `.curr_room`. 

Let's stylize all of this. Let's make the `.curr_room` stand out more than the other `.room`s because it's the room that the player is currently in. Let's also make `.win` and `.lose` hidden: they will appear only when the player wins or loses the game.

```css
.line {
    display: flex;
    flex-direction: row;
    flex-wrap: wrap;
}

.room {
    width: 30px;
    height: 30px;
    border: 1px dotted rgb(100, 10, 10);
    margin: 3px;
    display: flex;
    flex-direction: row;
    justify-content: center;
    align-items: center;
}

.curr_room {
    background:rgb(240, 220, 180);
    border: 2px solid rgb(100, 10, 10);
}

.win, .lose {
    display: none;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    text-align: center;
}

.win h4, .lose h4 {
    line-height: 100%;
    margin: 0;
}
```

Let's add the controls too. Below the for loop in `game.html` add:

```html
<div class="controls">
    <div class="left"><i data-feather="arrow-left"></i></div>
    <div class="up_down">
        <div class="up"><i data-feather="arrow-up"></i></div>
        <div class="down"><i data-feather="arrow-down"></i></div>
    </div>
    <div class="right"><i data-feather="arrow-right"></i></div>    
</div>

<script>
    feather.replace()
</script>
```

They're simple controls, just arrow keys. Let's add some style:

```css
.controls {
    display: flex;
    flex-direction: row;
    align-items: center;
    justify-content: center;
    justify-items: center;
    align-content: center;
    padding: 10px 15px 10px 15px;
}

.up, .left, .down, .right {
    display: flex;
    flex-direction: column;
    flex-grow: 1;
    background: rgb(100, 10, 10);
    color: rgb(240, 220, 180);
    padding: 8px 24px 8px 24px;
    margin: 3px;
    text-align: center;
    cursor: pointer;
    font-size: small;
    align-items: center;
}
```

Let's see how it is now:

![Game screen](/images/Screenshot_3.png)

#### Step 4 - Interacting with the game using JavaScript and Socket.io

Now it's time to develop the interaction with the game. First, let's capture the actions of the player. Go to `main.js` and add:

```javascript
playing = true

$(document).ready(function(){

    const socket = io();

    //actions

    $('.left').click(function(){
        if (playing) socket.emit('next_pos', {'action':'left'})
    })

    $('.right').click(function(){
        if (playing) socket.emit('next_pos', {'action':'right'})
    })

    $('.up').click(function(){
        if (playing) socket.emit('next_pos', {'action':'up'})
    })

    $('.down').click(function(){
        if (playing) socket.emit('next_pos', {'action':'down'})
    })

})
```

We are emitting the event `next_pos` and specifying the `action`: `left`, `right`, `up` or `down` in the message. Now the flask app in the server will receive the event `next_pos` and the message. When this happens, the app needs to calculate the next position of the player, and send it back to the client. Go back to `app.py` and add:

```python
@socketio.on('next_pos')
def next_pos(message):
    if message['action'] == 'left':
        if 0 <= session['curr_pos'][0] - 1 < session['width']:
            session['curr_pos'][0] -= 1
    elif message['action'] == 'right':
        if 0 <= session['curr_pos'][0] + 1 < session['width']:
            session['curr_pos'][0] += 1
    elif message['action'] == 'up':
        if 0 <= session['curr_pos'][1] - 1 < session['height']:
            session['curr_pos'][1] -= 1
    elif message['action'] == 'down':
        if 0 <= session['curr_pos'][1] + 1 < session['height']:
            session['curr_pos'][1] += 1
    elif message['action'] == 'arrow':
        pass

    pos = session['curr_pos']
    
    socketio.emit('update_status', 
                {"x": pos[0],
                "y": pos[1]}, 
                room=session['sid'])
```

When the server receives the `next_pos` event, it will update the `curr_pos` variable in the user session and send it back to the client using `socketio.emit('update_status')` with the player's new coordinates in the message. Note the line `room=session['sid']`. This is extremely important. This line is saying that the message will be sent only to the user's room. If this line wasn't there, the event would be sent to every user, messing up with the game of players who were playing simultaneously. You can test this by removing this line and playing the game with two or more devices simultaneously.

Now go back to `main.js` and let's handle the `update_status` event. Inside the `$(document).ready()` add:

```javascript
socket.on('update_status', (data)=>{
        curr = $(".curr_room")
        curr.removeClass("curr_room")

        room = $(".room[x='" + data['x'] + "'][y='" + data['y'] + "']")
        room.addClass("curr_room")
    })
```

We are removing the `.curr_room` style from the old player's room and adding it to the new player's room. But how will the player know where's the pits, the Wumpus, or the treasure? Let's use the Feather icons to signalize this to the player. First, we need to go back to `app.py` and verify what is in the adjacency of the player. Inside `next_pos()`, after the computation of the new position let's add:

```python
pos = session['curr_pos']
    print(pos)

adj = []

pit = 0
wumpus = 0
treasure = 0

if session['dungeon'][pos[1]][pos[0]] == 1:
    pit = 2
    socketio.emit('lose', room=session['sid'])
elif session['dungeon'][pos[1]][pos[0]] == 2:
    wumpus = 2
    socketio.emit('lose', room=session['sid'])
elif session['dungeon'][pos[1]][pos[0]] == 3:
    treasure = 2
    socketio.emit('win', room=session['sid'])
else:
    try:
        adj.append(session['dungeon'][pos[1]-1][pos[0]])
    except:
        pass

    try:
        adj.append(session['dungeon'][pos[1]+1][pos[0]])
    except:
        pass

    try:
        adj.append(session['dungeon'][pos[1]][pos[0]-1])
    except:
        pass

    try:
        adj.append(session['dungeon'][pos[1]][pos[0]+1])
    except:
        pass

    if 3 in adj:
        treasure = 1

    if 2 in adj:
        wumpus = 1
        
    if 1 in adj:
        pit = 1
```

With this, we can track the player's adjacency. 

- If `pit = 1`, then there's a pit around

- If `wumpus = 1`, then the Wumpus is around

- If `treasure = 1`, then the treasure is around

- If `pit = 2`, then the player fell into a pit and lost the game

- If `wumpus = 2`, then the player was eaten by the Wumpus and lost the game

- If `treasure = 2`, then the player found the treasure and won the game

When the player loses or wins the game, we emit the `lose` or `win` event. We also need to pass what is in the player's adjacency to the client. Let's change the `socketio.emit('update_status')` in the end of `next_pos()` to:

```python
socketio.emit('update_status', 
                {"x": pos[0],
                "y": pos[1],
                "pit": pit,
                "wumpus": wumpus,
                "treasure": treasure}, 
                room=session['sid'])
```

The `next_pos()` should be looking like this:

```python
@socketio.on('next_pos')
def next_pos(message):
    if message['action'] == 'left':
        if 0 <= session['curr_pos'][0] - 1 < session['width']:
            session['curr_pos'][0] -= 1
    elif message['action'] == 'right':
        if 0 <= session['curr_pos'][0] + 1 < session['width']:
            session['curr_pos'][0] += 1
    elif message['action'] == 'up':
        if 0 <= session['curr_pos'][1] - 1 < session['height']:
            session['curr_pos'][1] -= 1
    elif message['action'] == 'down':
        if 0 <= session['curr_pos'][1] + 1 < session['height']:
            session['curr_pos'][1] += 1
    elif message['action'] == 'arrow':
        pass

    pos = session['curr_pos']
    adj = []
    pit = 0
    wumpus = 0
    treasure = 0

    if session['dungeon'][pos[1]][pos[0]] == 1:
        pit = 2
        socketio.emit('lose', room=session['sid'])
    elif session['dungeon'][pos[1]][pos[0]] == 2:
        wumpus = 2
        socketio.emit('lose', room=session['sid'])
    elif session['dungeon'][pos[1]][pos[0]] == 3:
        treasure = 2
        socketio.emit('win', room=session['sid'])
    else:
        try:
            adj.append(session['dungeon'][pos[1]-1][pos[0]])
        except:
            pass

        try:
            adj.append(session['dungeon'][pos[1]+1][pos[0]])
        except:
            pass

        try:
            adj.append(session['dungeon'][pos[1]][pos[0]-1])
        except:
            pass

        try:
            adj.append(session['dungeon'][pos[1]][pos[0]+1])
        except:
            pass

        if 3 in adj:
            treasure = 1

        if 2 in adj:
            wumpus = 1
        
        if 1 in adj:
            pit = 1
        

    socketio.emit('update_status', 
                {"x": pos[0],
                "y": pos[1],
                "pit": pit,
                "wumpus": wumpus,
                "treasure": treasure}, 
                room=session['sid'])
```

Now the client will know what is in the player's adjacency and we can add the icons to signalize this. Go back to `main.js` and let's handle this new data. Change the handling of `update_status` in `main.js` to:

```javascript
socket.on('update_status', (data)=>{
        curr = $(".curr_room")
        curr.removeClass("curr_room")

        room = $(".room[x='" + data['x'] + "'][y='" + data['y'] + "']")
        room.empty()
        if (data['pit'] == 2) {
            room.append("<i data-feather='x-circle'></i>")
        } else if (data['wumpus'] == 2) {
            room.append("<i data-feather='frown'></i>")
        } else if (data['treasure'] == 2) {
            room.append("<i data-feather='award'></i>")
        } else {
            if (data['pit'] == 1) {
                room.append("<i data-feather='wind'></i>")
            }
            if (data['wumpus'] == 1) {
                room.append("<i data-feather='alert-triangle'></i>")
            }
            if (data['treasure'] == 1) {
                room.append("<i data-feather='star'></i>")
            }
        }
        feather.replace()
        room.addClass("curr_room")
    })
```

We add the following icons:

- `wind` icon when there's a pit nearby (`data['pit'] == 1`)

- `alert-triangle` icon when the Wumpus is nearby (`data['wumpus'] == 1`)

- `star` icon when the treasure is nearby (`data['treasure'] == 1`)

- `award` icon when the player finds the treasure (`data['treasure'] == 2`)

- `frown` icon when the player is killed by Wumpus (`data['wumpus'] == 2`)

- `x-circle` icon when the player falls in a pit (`data['pit'] == 2`)

We need to handle the `win` and `lose` events too. Add this to `$(document).ready()`:

```javascript
    // win or lose  
    socket.on('win', (data)=>{
        $('.win').css("display", "flex")
        $('.win').show("fast")
        playing = false
    })

    socket.on('lose', (data)=>{
        $('.lose').css("display", "flex")
        $('.lose').show("fast")
        playing = false
    })
```

We show the `.lose` or `.win` div and set `playing` to `false`, making it impossible for the player to move.

Finally, let's just make it possible to move around the map using the keyboard by detecting the arrows keypress:

```javascript
// detecting arrow keys
$(document).keydown(function(e) {
    if (e.keyCode == 37) {
        //left
        if (playing) socket.emit('next_pos', {'action':'left'})
    } else if (e.keyCode == 38) {
        //up
        if (playing) socket.emit('next_pos', {'action':'up'})
    } else if (e.keyCode == 39) {
        //right
        if (playing) socket.emit('next_pos', {'action':'right'})
    } else if (e.keyCode == 40) {
        //down
        if (playing) socket.emit('next_pos', {'action':'down'})
    }
})
```

And that's it! The game is ready to play! Go on and test it. What do you think?

![Game round](/images/Screenshot_4.png)

#### Conclusion

In this tutorial, you learned about the Hunt the Wumpus games and their influence on Artificial Intelligence. We also developed a simple adaptation of the Hunt the Wumpus game using Python, Flask, HTML, CSS, JavaScript e Socket.io. You can see how it's possible to create games in a very easy way using these technologies. You can now explore the endless possibilities of applications that you can build with them! Also, feel free to increment the game with new functionalities and complexity!

#### References

- [Hunt the Wumpus - Wikipedia](https://en.wikipedia.org/wiki/Hunt_the_Wumpus)

- [AI | The Wumpus World Description - GeeksforGeeks](https://www.geeksforgeeks.org/ai-the-wumpus-world-description/)

- [Artificial Intelligence: A Modern Approach](https://www.amazon.com.br/Artificial-Intelligence-Approach-Stuart-Russell/dp/0134610997)

- [Flask](https://flask.palletsprojects.com/en/2.0.x/)

- [Socket.io](https://socket.io/)