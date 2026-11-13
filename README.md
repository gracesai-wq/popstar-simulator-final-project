# drafter-deployment-template-v1
Your first Drafter website!
from drafter import *
from dataclasses import dataclass
import random

set_website_title("Popstar Simulator")
set_site_information(
    author="Grace Saintil",
    description="Order snacks for very good dogs.",
    sources="Thanks to the Drafter docs and my brother.",
    planning=[],
    links=[]
)
hide_debug_information()
set_website_framed(False)

@dataclass
class Budget:
    #Stores the amount of money available in a specific budget
    Amount: int


@dataclass
class State:
    #Stores all information about the player's popstar and game progress
    popstar_name: str
    city: str
    genre: str
    talent_level: int
    energy_level: int
    money: list[Budget]
    popularity_level: int
    question_number: int
    music_visits: int
    concert_visits: int
    sosh_meed_visits: int
    choices: str


@route
def index(state: State) -> Page:
    #Displays the welcome page and allows the player to start the game
    return Page(state, [
        "Welcome to the Popstar Simulator!",
        "\n",
        "Click the Button Below to Get Started",
        "\n",
        Button("Create Your Popstar", "create_popstar")
    ])


@route
def game_over(state: State, message: str = "You failed as a popstar!") -> Page:
    #Displays the game over screen and explains why the player lost
    return Page(state, [
        "GAME OVER!",
        message,
        Button("Start Over", "restart")
    ])


@route
def restart(state: State) -> Page:
    #Resets all player information and starts a new game
    fresh_state = State( "", "", "", 0, 100,[Budget(1500), Budget(1500), Budget(1500)],75, 1, 0, 0, 0, "")
    return index(fresh_state)


@route
def win_screen(state: State) -> Page:
    #Calculates the player's final performance and displays their title
    total_money = state.money[0].Amount + state.money[1].Amount + state.money[2].Amount

    # Defines thresholds for high stats
    high_pop = state.popularity_level > 50
    high_energy = state.energy_level > 50
    high_money = total_money > 3000

    # Calculate ratios to compare stats fairly
    pop_ratio = state.popularity_level / 100
    energy_ratio = state.energy_level / 100
    money_ratio = total_money / 4500

    # Determines their title
    if high_pop and high_energy and high_money:
        title = "The All-Around Superstar!"
        message = "You balanced fame, fortune, and self-care. You completed the simulator with high marks across the board!"

    elif high_pop and (not high_energy or pop_ratio >= energy_ratio) and (not high_money or pop_ratio >= money_ratio):
        title = "The Popular"
        message = "You completed the simulator with a popularity level over 50!"

    elif high_energy and (not high_pop or energy_ratio >= pop_ratio) and (not high_money or energy_ratio >= money_ratio):
        title = "The Well-Rested!"
        message = "You completed the simulator with tons of energy to spare!"

    elif high_money:
        title = "The Thrifty!"
        message = "You completed the simulator with over $3,000 in the bank!"

    else:
        title = "The Underdog!"
        message = "You survived the music industry! Your stats weren't high, but you made it to the end."

    # Factors in Talent Level
    if state.talent_level >= 8:
        talent_note = "Plus, with a Talent Level of " + str(state.talent_level) + "/10"

    elif state.talent_level >= 5:
        talent_note = "With a solid Talent Level of " + str(state.talent_level) + "/10"

    else:
        talent_note = "Even with a low Talent Level of " + str(state.talent_level) + "/10"

    return Page(state, [
        "SIMULATOR COMPLETE! \n",
        title + "\n",
        message + "\n",
        talent_note,
        "Final Popularity: " + str(state.popularity_level) + "/100" + "\n",
        "Final Energy: " + str(state.energy_level) + "/100" + "\n",
        "Final Total Money: $" + str(total_money) + "\n",
        Button("Play Again", "restart")
    ])


@route
def create_popstar(state: State) -> Page:
    #Asks the player questions to create their popstar character
    if state.question_number == 1:
        return Page(state, [
            "Type your Popstar name here:",
            TextBox("popstar_name", state.popstar_name),
            Button("Next", "next_question")
        ])

    elif state.question_number == 2:
        return Page(state, [
            "Pick which city you're based in!",
            RadioButtonGroup("city", [
                "Los Angeles, California",
                "\n",
                "New York City, New York",
                "\n",
                "Houston, Texas",
                "\n",
                "Seoul, South Korea"
            ]),
            Button("Next", "next_question")
        ])

    elif state.question_number == 3:
        return Page(state, [
            "Pick what kind of music you want to make!",
            RadioButtonGroup("genre", [
                "Hyperpop",
                "\n",
                "Country",
                "\n",
                "K-Pop",
                "\n",
                "RnB"
            ]),
            Button("Next", "next_question")
        ])

    elif state.question_number == 4:
        num = random.randint(1, 10)
        state.talent_level = (num)

        return Page(state, [
            "Your talent level has been generated!",
            "Talent Level: " + str(state.talent_level),
            Button("Finish", "home")
        ])

    return Page(state, [
        "Something went wrong.",
        Button("Start Over", "restart")
    ])


@route
def next_question(state: State, popstar_name: str = "", city: str = "", genre: str = "") -> Page:
    #Saves the player's character choices and moves to the next question.
    if popstar_name:
        state.popstar_name = popstar_name

    if city:
        state.city = city

    if genre:
        state.genre = genre

    state.question_number += 1
    return create_popstar(state)


@route
def home(state: State) -> Page:
    #Displays the main menu and checks whether the player won or lost
    # Checks if energy reached 0 or below
    if state.energy_level <= 0:
        return game_over(state, "You ran out of energy and burned out!")

    # Checks if opularity reached 0 or below
    if state.popularity_level <= 0:
        return game_over(state, "Your popularity hit 0 and you faded into obscurity!")

    # Checks if all budgets reached 0 or below
    if state.money[0].Amount <= 0 and state.money[1].Amount <= 0 and state.money[2].Amount <= 0:
        return game_over(state, "You went completely broke across all budgets!")

    # Checks if player finished all activities
    if state.music_visits >= 3 and state.sosh_meed_visits >= 3 and state.concert_visits >= 3:
        return win_screen(state)
    # Returns home page
    return Page(state, [
        "Welcome to your Popstar Simulator, " + (state.popstar_name or "Popstar") + "!","\n",
        "Work on some music!",
        Button("Music", "music"),
        "\n",
        "Interact with your fans!",
        Button("Social Media", "sosh_meed"),
        "\n",
        "Plan a Concert",
        Button("Concert", "concert"),
        "\n",
        "View Your Stats",
        Button("Stats", "user_stats")
    ])


# Music

@route
def music(state: State) -> Page:
    # Displays music-related scenarios
    if state.music_visits == 0:
        return Page(state, [
            "Your producer wants you to work with a really terrible beat because \n she thinks you can make this work. What do you do?",
            RadioButtonGroup("choices", [
                "Use the beat and protect your producers feelings" + "\n",
                "Reject the beat and get on bad terms with your producer" + "\n",
                "Bring up another artist who you think the beat would suit" + "\n"
            ]),
            Button("Back to Home", "next_music")
        ])

    elif state.music_visits == 1:
        return Page(state, [
            "Your nose is ridiculously stuffed, but your label needs you to finish recording today. What do you do?",
            RadioButtonGroup("choices", [
                "Go in and finish the song sounding nasally" + "\n",
                "Wait until you feel better and postpone the release" + "\n",
                "Record a demo now and redo the vocals when you feel better" + "\n"
            ]),
            Button("Back to Home", "next_music")
        ])

    elif state.music_visits == 2:
        return Page(state, [
            "Fans are getting bored of your releases. What do you do?",
            RadioButtonGroup("choices", [
                "Release a song outside of your genre" + "\n",
                "Work on songwriting" + "\n",
                "Collab with another artist" + "\n"
            ]),
            Button("Back to Home", "next_music")
        ])

    else:
        return Page(state, [
            "You've completed your music activities!",
            Button("Back to Home", "home")
        ])


@route
def next_music(state: State, choices: str = "") -> Page:
    # Processes music choices and adjusts the player's stats
    state.choices = choices

    if state.music_visits == 0:
        if choices == "Use the beat and protect your producers feelings" + "\n":
            state.popularity_level += 2
            state.energy_level -= 5

        elif choices == "Reject the beat and get on bad terms with your producer" + "\n":
            state.popularity_level -= 5
            state.energy_level -= 2

        elif choices == "Bring up another artist who you think the beat would suit" + "\n":
            state.popularity_level += 5
            state.energy_level -= 3

    elif state.music_visits == 1:
        if choices == "Go in and finish the song sounding nasally" + "\n":
            state.popularity_level -= 3
            state.energy_level -= 8

        elif choices == "Wait until you feel better and postpone the release" + "\n":
            state.popularity_level -= 2
            state.energy_level += 5

        elif choices == "Record a demo now and redo the vocals when you feel better" + "\n":
            state.popularity_level += 2
            state.energy_level -= 3
            state.money[0].Amount -= 100

    elif state.music_visits == 2:
        if choices == "Release a song outside of your genre" + "\n":
            state.popularity_level += 8
            state.energy_level -= 8
            state.money[0].Amount -= 200

        elif choices == "Work on songwriting" + "\n":
            state.talent_level = min(10, state.talent_level + 1)
            state.energy_level -= 5

        elif choices == "Collab with another artist" + "\n":
            state.popularity_level += 6
            state.energy_level -= 8
            state.money[0].Amount -= 250

    state.music_visits += 1
    return home(state)


# Social Media


@route
def sosh_meed(state: State) -> Page:
    # Displays social media scenarios
    if state.sosh_meed_visits == 0:
        return Page(state, [
            "You are getting a TON of hate online for your newest release. What do you do??",
            RadioButtonGroup("sosh_choice", [
                "Make fun of your release with them" + "\n",
                "Post a passive aggressive video calling out your haters" + "\n",
                "Ignore it" + "\n"
            ]),
            Button("Back to Home", "next_sosh_meed")
        ])

    elif state.sosh_meed_visits == 1:
        return Page(state, [
            "Your controversial tweets from 2013 are resurfacing. What do you do??",
            RadioButtonGroup("sosh_choice", [
                "Apology video with tears" + "\n",
                "Justify your actions by saying it was a different time" + "\n",
                "Ignore it" + "\n"
            ]),
            Button("Back to Home", "next_sosh_meed")
        ])

    elif state.sosh_meed_visits == 2:
        return Page(state, [
            "How do you want to interact with your fans?",
            RadioButtonGroup("sosh_choice", [
                "Post about your merch" + "\n",
                "Go on live and chat about your upcoming releases" + "\n",
                "Don't interact and scroll instead" + "\n"
            ]),
            Button("Back to Home", "next_sosh_meed")
        ])

    else:
        return Page(state, [
            "You've completed your social media activities!",
            Button("Back to Home", "home")
        ])


@route
def next_sosh_meed(state: State, sosh_choice: str = "") -> Page:
    # Processes social media decisions and adjusts the player's stats
    state.choices = sosh_choice

    if state.sosh_meed_visits == 0:
        if sosh_choice == "Make fun of your release with them" + "\n":
            state.popularity_level += 8
            state.energy_level -= 5
            state.money[1].Amount -= 100

        elif sosh_choice == "Post a passive aggressive video calling out your haters" + "\n":
            state.popularity_level -= 10
            state.energy_level -= 8
            state.money[1].Amount -= 100

        elif sosh_choice == "Ignore it" + "\n":
            state.energy_level -= 2

    elif state.sosh_meed_visits == 1:
        if sosh_choice == "Apology video with tears" + "\n":
            state.popularity_level += 5
            state.energy_level -= 5

        elif sosh_choice == "Justify your actions by saying it was a different time" + "\n":
            state.popularity_level -= 12
            state.energy_level -= 3

        elif sosh_choice == "Ignore it" + "\n":
            state.popularity_level -= 5

    elif state.sosh_meed_visits == 2:
        if sosh_choice == "Post about your merch" + "\n":
            state.money[1].Amount += 200
            state.popularity_level += 3
            state.energy_level -= 2

        elif sosh_choice == "Go on live and chat about your upcoming releases" + "\n":
            state.popularity_level += 6
            state.energy_level -= 5

        elif sosh_choice == "Don't interact and scroll instead" + "\n":
            state.energy_level += 3

    state.sosh_meed_visits += 1
    return home(state)

# Concert

@route
def concert(state: State) -> Page:
    # Displays concert-related scenarios
    if state.concert_visits == 0:
        return Page(state, [
            "Someone front and center is holding a posterboard with a \n screenshot of one of your controversial tweets from 2013. What do you do?",
            RadioButtonGroup("concert_choice", [
                "Ignore it and continue performing" + "\n",
                "Passive aggressively address it" + "\n",
                "Start bawling on stage" + "\n"
            ]),
            Button("Back to Home", "next_concert")
        ])

    elif state.concert_visits == 1:
        return Page(state, [
            "Your backtrack stops working and you get caught lipsyncing mid concert. What do you do?",
            RadioButtonGroup("concert_choice", [
                "Tell your fans you're sick and your label forced you to perform anyways" + "\n",
                "Own it" + "\n",
                "Start actually singing and hope nobody noticed" + "\n"
            ]),
            Button("Back to Home", "next_concert")
        ])

    elif state.concert_visits == 2:
        return Page(state, [
            "A big venue allows you to perform, but you know the audience is typically very critical. \n This could make or break your career. What do you do?",
            RadioButtonGroup("concert_choice", [
                "Perform to the best of your abilities" + "\n",
                "Decline and perform at a venue you're comfortable with" + "\n",
                "Perform but joke a lot so if it's bad people think you weren't serious" + "\n"
            ]),
            Button("Back to Home", "next_concert")
        ])

    else:
        return Page(state, [
            "You've completed your concert activities!",
            Button("Back to Home", "home")
        ])


@route
def next_concert(state: State, concert_choice: str = "") -> Page:
    # Processes concert decisions and adjusts the player's stats
    state.choices = concert_choice

    if state.concert_visits == 0:
        if concert_choice == "Ignore it and continue performing" + "\n":
            state.energy_level -= 2

        elif concert_choice == "Passive aggressively address it" + "\n":
            state.popularity_level -= 5
            state.energy_level -= 5

        elif concert_choice == "Start bawling on stage" + "\n":
            state.popularity_level -= 3
            state.energy_level -= 8

    elif state.concert_visits == 1:
        if concert_choice == "Tell your fans you're sick and your label forced you to perform anyways" + "\n":
            state.popularity_level -= 8
            state.energy_level -= 3

        elif concert_choice == "Own it" + "\n":
            state.popularity_level += 8
            state.energy_level -= 4

        elif concert_choice == "Start actually singing and hope nobody noticed" + "\n":
            state.popularity_level += 5
            state.energy_level -= 10

    elif state.concert_visits == 2:
        if concert_choice == "Perform to the best of your abilities" + "\n":
            state.popularity_level += 12
            state.energy_level -= 12
            state.money[2].Amount -= 200

        elif concert_choice == "Decline and perform at a venue you're comfortable with" + "\n":
            state.popularity_level += 4
            state.energy_level -= 4
            state.money[2].Amount += 100

        elif concert_choice == "Perform but joke a lot so if it's bad people think you weren't serious" + "\n":
            state.popularity_level -= 4
            state.energy_level -= 6

    state.concert_visits += 1
    return home(state)

# Stats

@route
def user_stats(state: State) -> Page:
    #Displays the player's current popstar information
    return Page(state, [
        "Popstar Stats for " + (state.popstar_name or "Unknown") + "\n",
        "City: " + state.city + "\n",
        "Genre: " + state.genre + "\n",
        "Talent Level: " + str(state.talent_level) + "/10 \n",
        "Energy Level: " + str(state.energy_level) + "/100 \n",
        "Popularity Level: " + str(state.popularity_level) + "/100 \n",
        "Music Budget: $" + str(state.money[0].Amount) + "\n",
        "Social Media Budget: $" + str(state.money[1].Amount) + "\n",
        "Concert Budget: $" + str(state.money[2].Amount) + "\n",
        Button("Back to Home", "home")
    ])


start_server(
    State(
        "", "", "", 0, 100, [Budget(1500), Budget(1500), Budget(1500)], 75, 1, 0, 0, 0, ""
    ),
    port=8002
    # I had a really weird error and asked my brother, who majored in comp sci,
    # and he told me to change ports (screenshots of texts included in submission)
    #We also called after and he helped me out a little more
)
