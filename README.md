from tkinter import *
from random import randint

# ________________CLASSES_______________________

class Snake:
    def __init__(self):
        self.body_size = BODY_SIZE
        self.coordinate = []
        self.square = []

        for i in range(0, BODY_SIZE):
            self.coordinate.append([0, 0])

        for x, y in self.coordinate:
            square: int = canvas.create_rectangle(x, y, x+SPACE_SIZE, y+SPACE_SIZE, fill=snake_color, tag="snake")
            self.square.append(square)


class Food:
    def __init__(self):
        self.x = randint(0, (screen_width // SPACE_SIZE)-1) * SPACE_SIZE
        self.y = randint(0, (screen_height // SPACE_SIZE)-1) * SPACE_SIZE
        self.coordinate = [self.x, self.y]
        self.oval = canvas.create_oval(self.x, self.y, self.x + SPACE_SIZE, self.y + SPACE_SIZE, fill=food_color, tag="food")

    def move(self):
        canvas.delete(self.oval)
        self.x = randint(0, (screen_width // SPACE_SIZE) - 1) * SPACE_SIZE
        self.y = randint(0, (screen_height // SPACE_SIZE) - 1) * SPACE_SIZE
        self.coordinate = [self.x, self.y]
        self.oval = canvas.create_oval(self.x, self.y, self.x + SPACE_SIZE, self.y + SPACE_SIZE, fill=food_color, tag="food")


# ________________________Functions______________________________


def next_turn(Snake, food):
    global direction, score, paused, running

    if not paused:
        x, y = Snake.coordinate[0]

        if direction == "up":
            y -= SPACE_SIZE
        elif direction == "down":
            y += SPACE_SIZE
        elif direction == "left":
            x -= SPACE_SIZE
        elif direction == "right":
            x += SPACE_SIZE

        snake.coordinate.insert(0, [x, y])
        square = canvas.create_rectangle(x, y, x+SPACE_SIZE, y+SPACE_SIZE, fill=snake_color)
        snake.square.insert(0, square)


        if x == food.coordinate[0] and y == food.coordinate[1]:
            global score
            score += 1
            Label.config(text=f'Score:{score}')
            food.move()
        else:
            del snake.coordinate[-1]
            canvas.delete(snake.square[-1])
            del snake.square[-1]

    if check_game_over(snake):
        game_over()
    else:
        window.after(speed_game, lambda: next_turn(Snake, food))



def check_game_over(snake):
    x, y = snake.coordinate[0]

    if x < 0 or x > screen_width:
        return True
    if y < 0 or y > screen_height:
        return True

    for sq in snake.coordinate[1:]:
        if x == sq[0] and y == sq[1]:
            return True
    return False


def game_over():
    canvas.delete(ALL)
    canvas.create_text(canvas.winfo_width() / 2, canvas.winfo_height() / 2,
                      font=("Terminal", 50),
                      text="GAME OVER!", fill="Red")

 
def change_direction(new_dir):
    global direction
    if new_dir == "up":
        if direction != "down":
            direction = new_dir
    elif new_dir == "right":
        if direction != "left":
            direction = new_dir
    elif new_dir == "down":
        if direction != "up":
            direction = new_dir
    elif new_dir == "left":
        if direction != "right":
            direction = new_dir


def toggle_pause(event):
    global paused
    paused = not paused
    if paused:
        canvas.create_text(canvas.winfo_width() / 2, canvas.winfo_height() / 2,
                           font=("Arial", 30),
                           text="PAUSED", fill="yellow", tag="paused")
    else:
        canvas.delete("paused")


def restart_program():
    global score, direction, snake, food, paused, speed_game
    canvas.delete(ALL)
    score = 0
    Label.config(text=f"Score: {score}")
    direction = "down"
    snake = Snake()
    food = Food()
    speed_game = 200
    paused = False
    next_turn(snake, food)


def increase_speed(event):
    global speed_game
    if speed_game > 50:
        speed_game -= 50


def decrease_speed(event):
    global speed_game
    if speed_game < 800:
        speed_game += 50

# _______________________________________


snake_color = "#FFD300"
food_color = "#fd6a02"
background_color = "black"
SPACE_SIZE = 28
BODY_SIZE = 3
score = 0
screen_width = 500
screen_height = 500
direction = "down"
speed_game = 200
paused = False
running = True
# _______________________________________
window = Tk()

window.title('Snake Game')
window.resizable(False, False)

Label = Label(window, text=f'Score:{score}', font=('Arial', 30))
Label.pack()

canvas = Canvas(window, width=screen_width, height=screen_height, bg=background_color)
canvas.pack()

window.bind('<Up>', lambda event: change_direction("up"))
window.bind('<Down>', lambda event: change_direction("down"))
window.bind('<Left>', lambda event: change_direction("left"))
window.bind('<Right>', lambda event: change_direction("right"))
window.bind('<space>', toggle_pause)
window.bind('<Shift_L>', increase_speed)
window.bind('<Control_L>', decrease_speed)

restart = Button(window, text='Restart', font=("Arial", 18), background="#FF6FFF", command=restart_program)
restart.pack()

window.update()

window_width = window.winfo_width()
window_height = window.winfo_height()
screenwidth = window.winfo_screenwidth()
screenheight = window.winfo_screenheight()

x = int((screenwidth / 2 ) - (window_width / 2))
y = int((screenheight / 2) - (window_height / 2))
window.geometry('{}x{}+{}+{}'.format(window_width, window_height, x, y))



snake = Snake()

food = Food()
next_turn(snake, food)

window.mainloop()
