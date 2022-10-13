# parser-site
Parsing of game-site


import requests as R #импортируем ранее установленную библиотеку requests
from bs4 import BeautifulSoup as BS #импортируем элемент из ранее установленной библиотеки beautifulsoup4
from time import sleep #импортируем элемент из ранее установленной библиотеки time, чтобы делать перерывы между парсингом страниц и не перегружать сервер (необязательно)
import pandas as pd #импортируем ранее установленную библиотеку pandas для экспорта в csv

# Обучающая часть, в финальном коде используются эти же элементы, но с помощью цикла

#r = R.get("https://stopgame.ru/topgames?rating=izumitelno&p=1") #присваиваем переменной функцию с URL первой страницы с данными, которая извлекает весь html-код 1-й страницы
#print(r.text) #выводит весь html-код 1-й страницы

#soup = BS(r.text,"html.parser") #передаем библиотеке BeautifulSoup извлеченный код страницы по указанному ранее URL
#element = soup.find('div', class_ = 'item game-summary game-summary-horiz').find('div', class_ = 'score')
#находим нужный класс, в котором содержится наш первый элемент, и дальше спускаемся вниз по dom-дереву, последовательно извлекая его характеристики
#piece_of_link = soup.find('div', class_ = 'item game-summary game-summary-horiz').find('a', class_ = 'link').get('href')
#извлекаем кусок ссылки с помощью метода get
#link = 'https://stopgame.ru'+piece_of_link #добавляем домен
#print(link) #выводит извлеченную ссылку для первой игры на странице
#full_name = soup.find('div', class_ = 'item game-summary game-summary-horiz').find('a', class_ = 'link').get('href')
#name = full_name.partition('/')[2].partition('/')[2].partition('/')[0]
#название вытягивается с лишними символами, но это не самый лучший способ получить нормальное название (порядок символов в названии может быть разный), лучший способ - метод select, которій используется в финальном коде
#print(name) #выводит извлеченное и обработанное название первой игры на странице
#platforms = soup.find('div', class_ = 'item game-summary game-summary-horiz').find('div', class_ = 'game-spec').find('span', class_ = 'value').text
# извлечение платформ для первой игры на странице. Добавляем функцию text вконце для преобразования объекта bs4 в текст
# метод find (в отличие от findall) находит только первый элемент класса game-spec, он нам и нужен
#print(platforms)
#jenre = soup.find('div', class_ = 'item game-summary game-summary-horiz').findAll('div', class_ = 'game-spec')[1].find('span', class_ = 'value').text
# функция findall находит все, забираем только второй элемент с жанром первой игры на странице
#print(jenre)
#date = soup.find('div', class_ = 'item game-summary game-summary-horiz').findAll('div', class_ = 'game-spec')[2].find('span', class_ = 'value').text
# функция findall находит все, забираем только третий элемент с датой выхода первой игры на странице
#print(date)
#rate= soup.find('div', class_ = 'item game-summary game-summary-horiz').find('div', class_ = 'score').find('div', class_ = 'value colored-bg-green').text
#print(rate)

#Всю верхнюю часть можно пропустить, она нужна только для поэтапного разбора алгоритма

data = []
# создаем пустой список, куда будем добавлять данные

for p in range (1,3): #тут 2 страницы
   print(p) #выводит номер страницы, которую парсит
   url = f'https://stopgame.ru/topgames?rating=izumitelno&p={p}' 
   #url = f'https://stopgame.ru/games?p={p}'
   # переписываем url - вместо номера стр. счетчик цикла в фигурных скобках {p}, чтобы собирать со всех страниц сайта, вывод в строчку (f'')
   r = R.get(url)
   sleep(3) # после первого запроса код ждет 3с.
   soup = BS(r.content, "html.parser")#передаем библиотеке BeautifulSoup извлеченный код страницы по указанному ранее URL, можно использовать как метод text, так и content

   games = soup.findAll('div', class_ = 'item game-summary game-summary-horiz')
   #находим все игры на одной странице с 1-м в иерархии классом
   #print(len(games))#30 игр на одной странице

    for game in games:
           name = game.select('.item .caption > a')[0].text
# извлекает НАЗВАНИЕ ссылки ПОД тегом "а". Важны пробелы. Сначала извлекает список, к которому невозможно применить ф-ю text, поэтому важно вытянуть содержание списка в текстом формате, а затем применять ф-ю
           link = 'https://stopgame.ru' + game.find('a', class_='link').get('href')
           platforms = game.find('div', class_='game-spec').find('span', class_='value').text
           jenre = game.findAll('div', class_='game-spec')[1].find('span', class_='value').text
           rate = game.find('div', class_='score').text.strip()
           date = game.findAll('div', class_='game-spec')[2].find('span', class_='value').text
   # цикл для извлечения этих 30 значений,
   # ('div', class_ = 'item game-summary game-summary-horiz') заменяем на game (итератор цикла), т.к. мы присвоили этот путь переменной games
           data.append([name,link,platforms,jenre,rate,date])
   #заполняем список
print(data)

header = ['name','link','platforms','jenre','rate','date']
#создаем переменную с колонками будущей таблицы
df = pd.DataFrame(data, columns=header) #pandas
#создаем таблицу
df.to_csv('C:/Users/Ioan/Desktop/Parsing_games.csv', sep = ';', encoding = 'utf8') #выгружаем в csv

