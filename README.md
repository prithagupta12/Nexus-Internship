# Nexus-Internship
import re
import random
import json
from difflib import get_close_matches

ADVICE = "Plan a good career with Machine Learning!"

def load_intents(file_path: str)-> dict:
    with open(file_path, 'r') as file:
        data: dict = json.load(file)
    return  data
def save_intents(file_path: str, data: dict):
    with open(file_path, 'w') as file:
        json.dump(data, file, indent=2)
def find_best_match(user_question: str, questions: list[str]) -> str | None:
    matches: list = get_close_matches(user_question, questions, n=4, cutoff=0.8)
    return matches[0] if matches else None
def get_answer_for_question(question: str, intents: dict) -> str | None:
    for q in intents["questions"]:
        if q["question"] == question:
            return q["answer"]
def chat_bot():
    intents: dict = load_intents('intents.json')
    
    while True:
        user_input: str = input('You: ')
        if user_input.lower() == 'quit':
            break
        best_match: str | None = find_best_match(user_input, [q["question"] for q in intents["questions"]])
        if best_match:
            answer: str = get_answer_for_question(best_match, intents)
            print(f'Bot: {answer}')
        else:
            print('Bot: I don\'t know the answer. Can you teach me?')
            new_answer: str = input('Type the answer or "skip" to skip: ')
            
            if new_answer.lower() != 'skip':
                intents["questions"].append({"question": user_input, "answer": new_answer})
                save_intents('intents.json', intents)
                print('Bot: Thank you! I learned a new response!')


def unknown():
    response = chat_bot()
    return response

def greet_user():
    print("Welcome in ChitChat Buddy!")

def message_probability(user_message, recognisedWords, single_response=False, required_words=[]):
    messageCertainty = 0
    hasrequiredwords = True

    for word in user_message:
        if word in recognisedWords:
            messageCertainty += 1              # Counts num of words present in each predefined message

    percentage = float(messageCertainty) / float(len(recognisedWords))    # Calculates the percent of recognised words in a user message

    for word in required_words:
        if word not in user_message:
            hasrequiredwords = False                   # Checks the required words are in the string
            break

    if hasrequiredwords or single_response:
        return int(percentage * 100)
    else:
        return 0

def check_all_messages(message):
    highest_prob_list = {}

    # Simplifies response creation / adds it to the dict
    def response(bot_response, list_of_words, single_response=False, required_words=[]):
        nonlocal highest_prob_list
        highest_prob_list[bot_response] = message_probability(message, list_of_words, single_response, required_words)

    # Responses
    response('Hello!', ['hello', 'hi', 'hey', 'sup', 'heyo'], single_response=True)
    response('See you!', ['bye', 'goodbye'], single_response=True)
    response('I\'m doing fine, and you?', ['how', 'are', 'you', 'doing'], required_words=['how'])
    response('You\'re welcome!', ['thank', 'thanks'], single_response=True)
    response('Thank you!', ['i', 'love', 'this', 'code'], required_words=['code', 'love'])

    # Long responses
    response(ADVICE, ['give', 'advice'], required_words=['advice', 'give'])
    

    best_match = max(highest_prob_list, key=highest_prob_list.get)

    return unknown() if highest_prob_list[best_match] < 1 else best_match

def get_response(user_input):
    split_message = re.split(r'\s+|[,;?!.-]\s*', user_input.lower())
    response = check_all_messages(split_message)
    return response

def goodbye_user():
    print("Good bye! Have a nice day:) If you want any help again, feel free to ask.")

# starting of main function

greet_user()
print('Bot: What is your name?')
name = input('You: ')
print('Bot: How old are you?')
age = input('You: ')
print('Bot: What is your ID?')
ID = input('You: ')
print(f"Your name is {name}, you are {age} years old and your ID is {ID}\n")
print(f"Welcome {name}. Now you can ask me anything! Start with greeting:)")

while True:
    user = input('You: ')
    if(user == 'You: bye'  or user == 'You: Bye' or user == 'You: ok bye'):
        goodbye_user()                                                       
        break
    res = get_response(user)
    print('Bot: ' + res )
    
