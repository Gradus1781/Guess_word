# Guess_word
# -*- coding: utf-8 -*-
from random import choice
from re import search
from time import sleep
from glob import glob

PATH_C = "C:/Users"  # где ищем
FILENAME = "world_word_rus" + ".txt"  # что ищем


def find_file(searchpath, name):
    while not glob(searchpath + name):
        searchpath += "*/"
        if len(searchpath) > 10:
            return False
    return glob(searchpath + name)


def open_file():
    words = ""
    try:
        absolute_path = str(find_file(PATH_C, FILENAME))[2:-2]
        with open(absolute_path, encoding='utf-8') as file:
            for line in file:
                words += line
    except FileNotFoundError as f:
        print("Поместите файл на диск С в папку Users.")
        raise f
    words = words.split(", ")
    return words


all_words = open_file()


def get_word(lst):
    # Выбираем случайное слово из списка в качестве загаданного.
    return choice(lst).upper()


def display_hangman(tries):
    stages = [
        # финальная стадия голова, торс, обе руки, обе ноги, обе ступни
        '''
           --------
           |      |
           |      O
           |     \\|/
           |      |
           |    _/ \\_
           -
        ''',
        # голова, торс, обе руки, обе ноги, одна ступня
        '''
           --------
           |      |
           |      O
           |     \\|/
           |      |
           |    _/ \\
           -
        ''',
        # голова, торс, обе руки, обе ноги
        '''
           --------
           |      |
           |      O
           |     \\|/
           |      |
           |     / \\
           -
        ''',
        # голова, торс, обе руки, одна нога
        '''
           --------
           |      |
           |      O
           |     \\|/
           |      |
           |     / 
           -
        ''',
        # голова, торс, обе руки
        '''
           --------
           |      |
           |      O
           |     \\|/
           |      |
           |      
           -
        ''',
        # голова, торс и одна рука
        '''
           --------
           |      |
           |      O
           |     \\|
           |      |
           |     
           -
        ''',
        # голова и торс
        '''
           --------
           |      |
           |      O
           |      |
           |      |
           |     
           -
        ''',
        # голова
        '''
           --------
           |      |
           |      O
           |    
           |      
           |     
           -
        ''',
        # начальное состояние
        '''
           --------
           |      |
           |      
           |    
           |      
           |     
           -
        '''
    ]
    return stages[tries]


def answer_to_the_question() -> bool:
    while True:
        answer = input(">>> ").lower()
        if answer == "да":
            return True
        elif answer == "нет":
            return False
        else:
            continue


def play(word):
    word1 = word[:]  # делаем копию загаданного слова, т.к. начальное слово мы будем менять
    word = list(word)
    word_completion = list('_' * len(word))  # список, содержащий символы _ на каждую букву задуманного слова
    guessed_letters = list()  # список уже названных букв
    guessed_words = list()  # список уже названных слов
    tries = 8  # количество попыток
    print("Открыть крайние буквы?(да, нет)")
    if answer_to_the_question():
        word_completion[0], word_completion[-1] = word1[0], word1[-1]
        word[0] = word[-1] = '_'
        guessed_letters.append(word1[0])
        guessed_letters.append(word1[-1])
        print("Подготавливаем применённые настройки...")
        sleep(2)
        print("Внимание! Если вам известны крайние буквы, это не значит, что они не могут повторяться в слове!")
        sleep(3)
    print("Всего у вас будет 8 попыток. Текущее состояние изображено ниже:")
    print(display_hangman(tries))
    sleep(1)
    print(f"Длина этого слова составляет {len(word)} букв. Текущее его состояние указано ниже:")
    print(*word_completion)
    sleep(1)
    print("Попробуйте угадать загаданное слово либо же угадывая его по буквам.")
    while True:
        answer = input("Ваше предположение: ").upper()
        result = search(r"[A-Z]", answer)
        if result is not None:
            print("В слове не должно быть английских букв.")
        elif not answer:
            print("Вы должны что-нибудь ввести.")
        elif not answer.isalpha():
            print("Спецсимволы и цифры в ответе недопустимы.")
        elif len(answer) > 1:
            if answer == word1:
                print(f"Поздравляем! Загаданным словом было {word1}. Вы угадали!")
                break
            elif answer != word1:
                if answer not in guessed_words:
                    guessed_words.append(answer)
                    tries -= 1
                    print(f"Неверно! Вы не угадали слово! Осталось попыток: {tries}")
                    print(display_hangman(tries))
                    print(*word_completion)
                else:
                    print("Это слово вы уже называли")
                    print("Названные слова: ", *guessed_words)
        elif len(answer) == 1:
            if answer in word:
                how_much_alphas_in_word = word.count(answer)
                for _ in range(how_much_alphas_in_word):
                    next_index = word.index(answer)
                    word_completion[next_index] = answer
                    word[next_index] = '_'
                guessed_letters.append(answer)
                print(*word_completion)
            elif answer not in guessed_letters:
                guessed_letters.append(answer)
                tries -= 1
                print(f"Такой буквы в этом слове нет! Осталось попыток: {tries}")
                print(display_hangman(tries))
                print(*word_completion)
            else:
                print("Вы уже называли эту букву")
                print("Названные буквы: ", *sorted(guessed_letters))
        if "_" not in word_completion:
            print(f"Поздравляем! Вы угадали слово {word1}!!!")
            break
        if tries == 0:
            print(f"Вы проиграли! Загаданное слово было: {word1} =(")
            break


def main_game():
    while True:
        hidden_word = get_word(all_words)  # Получаем загаданное слово
        play(hidden_word)
        print("Спасибо за игру!")
        print('Хотите сыграть ещё?(да, нет): ')
        if answer_to_the_question():
            continue
        else:
            print("Вcего доброго! Возвращайтесь к нам ещё!")
            break


print("Давайте играть в угадайку слов!")
main_game()
