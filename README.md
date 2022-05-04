<a id="anchor"></a>
# Приложение с графическим пользовательским интерфейсом для инвентаризации книг
___
Содержание:
* [1. Программа в действии](#screenshot)
* [2. Работа над Frontend файлом](#frontend)
* [3. Работа над Backend файлом](#backend)
* [4. Коннект фронтенда с бэкендом](#connect)
___

* Приложение создано с использованием _GUI_ библиотеки _Tkinter_ и библиотеки _SQLite_ для работы с базой данных. 

* Приложение имеет 4 текстовых поля для ввода данных о книге _(Название, Автор, Год и ISBN уникальный идентификационный номер)_ и сохранения их в базе.

* С помощью кнопок приложения, пользователь может добавить новую книгу в базу, изменить данные о книге, найти книгу по одному из четырех параметров и вывести информацию на экран, посмотреть все записи о книгах, удалить запись.

* __В программе так же реализован объектно-ориенированный подход. Код программы с использованием методов ООП см. в файле: bookstore/_frontend_oop.py_ и _bookstore/backend_oop.py_.__

<a id="screenshot"></a>
### Программа в действии
![IMG_1845](https://user-images.githubusercontent.com/97599612/166640640-5213852a-886b-4a1e-b014-783de580740f.JPG)

## Создание:

<a id="frontend"></a>

___Начнем с создания Frontend файла. Создадим графический пользовательский интерфейс: главное окно, кнопки, поля ввода, скроллер, которые пока не будут выполнять никаких действий.___
* _Полный код см. в файле bookstore/frontend.py_

#### 1. Создаем главное окно.
```
window = Tk()
window.mainloop()
```

#### 2. Создаем виджеты.

* Метки:
```
l1=Label(window,text="Title")
l1.grid(row=0,column=0) # используем метод grid
```

* Текстовые поля ввода:
```
title_text = StringVar()
e1 = Entry(window, textvariable=title_text)
e1.grid(row=0,column=1)
```

* Список (Listbox):
```
list1 = Listbox(window, height=6, width=35)
list1.grid(column=0, row=2,rowspan=6,columnspan=2)
```

* Скроллер:
```
sb1 = ttk.Scrollbar(window, orient=VERTICAL, command=list1.yview)
sb1.grid(column=2, row=2,rowspan=6)

list1.configure(yscrollcommand=sb1.set) # применение метода configure к списку listbox  
sb1.configure(command=list1.yview)
```

* Кнопки:
```
b1 = Button(window, text='View all', width=12)
b1.grid(row=2,column=3)
```

[Вверх](#anchor)


<a id="backend"></a>

___Создадим Backend файл с функциями просмотра, поиска, добавления, удаления, редактирования записей.
То есть, определим функции для каждой из созданных кнопок. Задействуем бибилиотеку SQLite3 для работы с базой данных и вызова данных на наш пользовательский интерфейс.___
* _Полный код см. в файле bookstore/backend.py_

#### 1. Создаем базу данных и подключаемся к ней.
```
def connect():
    conn = sqlite3.connect("books.db")
    cur = conn.cursor()
    cur.execute("CREATE TABLE IF NOT EXISTS book (id INTEGER PRIMARY KEY, title text, author text, year integer, isbn integer)")
    conn.commit()
    conn.close()
```

#### 2. Функция вставки данных.
```
def insert(title, author, year, isbn):
    cur = conn.cursor()
    cur.execute("INSERT INTO book VALUES (NULL, ?, ?, ?, ?)",(title, author, year, isbn))
```

\# _NULL_ означает, что _id_ будет создан автоматически.

\# 4 знака ? для каждого из аргументов функции.

\# Далее передаем параметры _(title, author, year, isbn)_ в качестве второго параметра функции _execute_.


#### 3. Функция просмостра записей.
```
def view():
    cur.execute("SELECT * FROM book")
```

#### 4. Функция поиска.
```
def search(title="", author="", year="", isbn=""):
    conn = sqlite3.connect("books.db")
    cur = conn.cursor()
    cur.execute("SELECT * FROM book WHERE title=? OR author=? OR year=? OR isbn=?",(title, author, year, isbn))
    rows = cur.fetchall()
    conn.close()
    return rows
```

\# Задействован _OR_ поиск. Это значит, что пользователь может ввести имя автора, год, назавание, или номер книги _(title, author, year, isbn)_, либо все параметры вместе. 
Допустим, если введен только год написания, то будут показаны все книги, вышедшие в указанном году. 

\# _(title="", author="", year="", isbn="")_ Пустые строки в качестве значений нужны, чтобы не возникала ошибка в случае, если будет введен аргумент только для одного из параметров, а оставшиеся параметры останутся без значений.


#### 5. Функция удаления записей.
```
def delete(id):
```

\# Функция принимает _id_ в качестве параметра для удаления из базы данных строки с данным _id_.


#### 6. Функция обновления записей.
```
def update(id, title, author, year, isbn):
```

#### 7. Функция проверки нахождения книги в базе.
```
def exists(title="", author="", year="", isbn=""):
    cur.execute("SELECT * FROM book WHERE title=? AND author=? AND year=? AND isbn=?",(title, author, year, isbn))
```

\# Функция такая же, как и функция _search_, только вместо _OR_ поиска задействован _AND_ поиск.


[Вверх](#anchor)


<a id="connect"></a>

___Связываем Фронтенд с Бэкендом___
* _Полный код см. в файле bookstore/frontend.py_
#### 1. Кнопка View all
```
b1 = Button(window, text='View all', width=12, command=view_command)
b1.grid(row=2,column=3)
```

\# Функция _view_command_ нужна чтобы получать данные функции _view_ файла _backend.py_ и вставлять эти данные в список _Listbox_ в _GUI_.

* Используем цикл _for_ для итерации списка кортежей и вывода их в качестве строк в _Listbox_:
```
def view_command():
    list1.delete(0, END)  # удаляет все записи из Listbox прежде чем выводить данные
    for row in backend.view():
        list1.insert(END, row)
```

#### 2. Кнопка Search entry
```
def search_command():
    list1.delete(0, END)
    for row in backend.search(title_text.get(), author_text.get(), year_text.get(), isbn_text.get()):
        list1.insert(END, row)
```

\# Параметры для функции _search_ берем из виджетов текстовых полей ввода.

\# Метод _.get()_ нужен так как _title_text_ или _author_text_ типы данных _StringVar()_, а не просто строка.


#### 3. Кнопка Add entry
```
def add_command():
    backend.insert(title_text.get(), author_text.get(), year_text.get(), isbn_text.get())
    list1.delete(0, END)
    list1.insert(END, (title_text.get(), author_text.get(), year_text.get(), isbn_text.get()))
```

#### 4. Кнопка Delete selected

* Функция _delete_ принимает _id_ в качестве аргумента и удаляет строку с соответсвующим _id_ из базы данных.

* Соответсвенно, когда пользователь выбирает одну из строк в списке _Listbox_, мы должны получить _id_ выбранной строки и передать его параметром в функцию _delete_ файла _backend.py_.


> list1.bind('<<ListboxSelect>>', get_selected_row)

Метод _bind_ привязывает функцию к виджету _Listbox_

* Далее определим функцию _get_selected_row_ в начале скрипта.
```
def get_selected_row(event):
    try:
        global selected_tuple  # глобальная переменная для использования в функции delete_command
        index=list1.curselection()[0]   # получаем индекс выбранной строки
        selected_tuple=list1.get(index)   # получаем кортеж со значениями строки
```

* Передадим _id_ выбранного кортежа/строки в функцию _delete_ файла _backend.py_.
```
def delete_command():
    backend.delete(selected_tuple[0])
```

* Для заполнения текстовых полей элементами кортежа, используем функцию _insert_ виджета _entry_.
```
    e1.delete(0,END)  # удаляет данные из текстового поля
    e1.insert(END,selected_tuple[1])  # selected_tuple[1] - соответсвенно название книги
    e2.delete(0,END)
    e2.insert(END,selected_tuple[2])  # selected_tuple[2] - автор книги
```

* Функция _get_selected_row_ теперь выглядит так:
```
def get_selected_row(event):
    try:
        global selected_tuple
        index=list1.curselection()[0]
        selected_tuple=list1.get(index)
        e1.delete(0,END)
        e1.insert(END,selected_tuple[1])
        e2.delete(0,END)
        e2.insert(END,selected_tuple[2])
        e3.delete(0,END)
        e3.insert(END,selected_tuple[3])
        e4.delete(0,END)
        e4.insert(END,selected_tuple[4])
    except IndexError:
        pass
```

\# Обработка исключений _try except_ нужна, чтобы не возникала ошибка _IndexError_, в случае, если пользователь кликнет по пустому списку _Listbox_. 


#### 5. Кнопка Update selected
```
def update_command():
    if len(exists_command())>0:
        list1.delete(0,END)
        list1.insert(END,"ERROR! :: Book Already Exists")
    else:
        backend.update(selected_tuple[0], title_text.get(), author_text.get(), year_text.get(), isbn_text.get())
```

\# _if else_ оператор нужен на случай если книга уже находится в базе. Будет выдана ошибка: Такая книга уже есть!


#### 6. Кнопка Close
```
b6 = Button(window, text='Close', width=12, command=window.destroy)
```

[Вверх](#anchor)