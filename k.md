## Работа с python

Первым делом установим python выполнив ряд команд ```sudo apt update```, как обычно проверили и обновились. ```sudo apt install build-essential zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev libssl-dev libsqlite3-dev libreadline-dev libffi-dev curl libbz2-dev```, установили зависимости, ```wget https://www.python.org/ftp/python/3.9.1/Python-3.9.10.tgz```, скачали исходник, ```tar -xf Python-3.9.10.tgz```, распаковали архив с исходником, ```cd Python-3.9.10```, зашли в распакованный архив и запусчкаем оптимизацию двоичного кода командой ```./configure --enable-optimizations```, запускаем процесс сборкии ```make -j 2```. Теперь же мы устанавливаем сами двоичные файлы ```sudo make altinstall``` и проверяем версию ```python3.9 --version```. Для удобства мы также установили _pycharm_ простой командой ```sudo snap install pycharm-community --classic```. В дальнейшем мы установили пакетный менеджер для Python, "Pip" ```sudo apt install python3-venv python3-pip``` и библиотеку для работы с PostgreSQL "psycopg2" командой ```pip install psycopg2-binary```.

### Первый опыт 

Первым делом мы проверили работу psycopg2 написав простой код в двух файлах, где первый файл конфигураци "config" для удобсва:
```
host = "127.0.0.1"
user = "postgres"
password = "clamav"
db_name = "virus"
```
Далее попробуем создать таблицу используя уже второй файл, который мы назвали "SQL":

```
import psycopg2
from config import host, user, password, db_name

# пробуем подключиться к PostgreSQL
try:
    connection = psycopg2.connect(
        host=host,
        user=user,
        password=password,
        database=db_name
    )
# разрешаем автоподключение и узнаем версию PostgreSQL
    connection.autocommit = True
    with connection.cursor() as cursor:
        cursor.execute(
            "SELECT version();"
        )
        print(f"Server version: {cursor.fetchone()}")
# подключаемя к базеданных и создаем таблицус задаными параметрами, выврдим сообщение при удачном создании
    with connection.cursor() as cursor:
            cursor.execute(""""CREATE TABLE scan(id serial PRIMARY KEY, folder varchar(8000), data char(20));"""))
            print("[INFO] Table created successfully")
# вывод возможных ошибок и закрытие соединения
except Exception as _ex:
    print("[INFO] Error while working with PostgresSQL", _ex)
finally:
    if connection:
        connection.close()
        print("[INFO] PostgresSQL connection closed")
```
Далее мы при помощи команды консоли ```clamscan -i /home/kal/Загрузки/ -l scan.log / & ``` просканировали папку "Загрузки" и создали лог файл, который имел вид:

```

-------------------------------------------------------------------------------

/home/kal/Загрузки/eicar_com.zip: Win.Test.EICAR_HDB-1 FOUND
/home/kal/Загрузки/eicar.com: Win.Test.EICAR_HDB-1 FOUND
/home/kal/Загрузки/pipe/non_virus.com: Win.Test.EICAR_HDB-1 FOUND
/home/kal/Загрузки/pipe/mb_virus.zip: Win.Test.EICAR_HDB-1 FOUND
/home/kal/Загрузки/postgresql/molly: Win.Test.EICAR_HDB-1 FOUND
/home/kal/Загрузки/postgresql/cian: Win.Test.EICAR_HDB-1 FOUND
/home/kal/Загрузки/postgresql/alla/misa.zip: Win.Test.EICAR_HDB-1 FOUND
/home/kal/Загрузки/postgresql/alla/inaros.com: Win.Test.EICAR_HDB-1 FOUND

----------- SCAN SUMMARY -----------
Known viruses: 8615055
Engine version: 0.104.2
Scanned directories: 4
Scanned files: 13
Infected files: 8
Data scanned: 38.55 MB
Data read: 81.73 MB (ratio 0.47:1)
Time: 147.589 sec (2 m 27 s)
Start Date: 2022:05:04 07:52:16
End Date:   2022:05:04 07:54:44

```
Тут нас интересуют название и пути найденых файлов и время когда они были найдены, для их выделения и записи мы написали "парсер логов", который имел вследующий вид:

```
# читаем логи
file = open('/home/kal/scan.log','r')
# создаем и файл vali.py, в который запишем результаты "парсинга" в виде вложеных списков, а именно [(folder, data),(folder, data),(folder, data),...(folder, data),]
my_file = open("vali.py", "w+")
my_file.write("values = [\n")
# ищем строку с датой окончания и в переменную Dates записываем значения данной строки с 12 по 31 символ, а именно "2022:05:04 07:54:44"
for line in file:
    if "End Date:" in line:
        Dates = line[12:31]
Dates = Dates
file.close()
my_file.close()
# ищем все найденные файлы, по ключевому слову "FOUND" и отсекавем все что идет после названия файла
file = open('/home/kal/scan.log','r')
my_file = open("vali.py", "a")
for line in file:
    if "FOUND" in line:
        vali = (line.split(': ')[0])
# записываем в файл "vali.py"
        my_file.write("("+"'"+vali+"'"+", "+"'"+Dates+"'"+"),\n")
my_file.write("]")
file.close()
my_file.close()
```
Полученный результат парсинга в файле "vali.py" выглядел так:
```
values = [
('/home/kal/Загрузки/eicar_com.zip', '2022:05:04 07:54:44'),
('/home/kal/Загрузки/eicar.com', '2022:05:04 07:54:44'),
('/home/kal/Загрузки/pipe/non_virus.com', '2022:05:04 07:54:44'),
('/home/kal/Загрузки/pipe/mb_virus.zip', '2022:05:04 07:54:44'),
('/home/kal/Загрузки/postgresql/molly', '2022:05:04 07:54:44'),
('/home/kal/Загрузки/postgresql/cian', '2022:05:04 07:54:44'),
('/home/kal/Загрузки/postgresql/alla/misa.zip', '2022:05:04 07:54:44'),
('/home/kal/Загрузки/postgresql/alla/inaros.com', '2022:05:04 07:54:44'),
]
```
values является переменной типа list(список), далее полученные данный мы записываем в БД используя следующий код:
```
import psycopg2
from config import host, user, password, db_name
from vali import values

try:
    connection = psycopg2.connect(
        host=host,
        user=user,
        password=password,
        database=db_name
    )
    connection.autocommit = True
    with connection.cursor() as cursor:
        cursor.execute(
            "SELECT version();"
        )
        print(f"Server version: {cursor.fetchone()}")
    with connection.cursor() as cursor:
# тут мы говорим вставить много данных в таблицу "scan" в столбцы "folder" и "data", взяв значения из переменной "values", импортируемой из файла "vali.py"
            cursor.executemany("INSERT INTO scan (folder, data) VALUES(%s,%s)", values)
            print("[INFO] Data was successfully inserted")
except Exception as _ex:
    print("[INFO] Error while working with PostgresSQL", _ex)
finally:
    if connection:
        connection.close()
        print("[INFO] PostgresSQL connection closed")
```
Так себе вариант но рабочий, мы сначала через консоль запускаем ClamAV с записью логов, потом же парсим эти логи в промежуточный файл и записываем в БД используется всего 5 файлов: config.py, scan.log, parser.py, vali.py, sql.py, что не есть хорошо, и если мы могли исбавиться от первого файла то от остальных на данном этапе мы не могли распрощаться.
### Второй опыт

В этот раз мы попробуем распрощаться со всеми не нужными файлами, оставив только sql.py как основной и config.py для удобства.
В этот раз у нас не будет файла scan.log, значит данные нужно брать из потока, vali.py мы использоавли только для того чтобы автоматически формировать переменную нужного формата, мы избавимся от этого костыля, что долгое время не могли сделать.

```
import subprocess
import psycopg2
from config import host, user, password, db_name
from ast import literal_eval

# начало формирования переменной list типо char
list = '('
print('input the path to the folder or file')
# просим указать путь и формируем команду для консоли
cmd = "clamdscan -i " + input()
# запускаем подпроцес под названием "process" который будет выполнять сканирование зааднного файла или папки, при этом будет произведена
# запись вывода потока данных stdout и потока ошибок stderr, при чем данные будут в байтовом виде. stdout и stderr это стандартные потоки
process = subprocess.Popen(cmd,shell=True,stdin=None,stdout=subprocess.PIPE,stderr=subprocess.PIPE)
result = process.stdout.readlines()
error = process.stderr.readlines()
# если поток ошибок не нулевой, то есть при сканировании была ошибка, мы выводим эту ошибку и завершаем дальнейшие манипуляции 
if len(error)!= 0:
    print(error)
    exit()
# построчно считывааем поток данных stdout декодировав стандарным utf-8 в поисках даты и дальнейшем записью в переменную
for line in result:
   data = line.decode("utf-8")
   if "End Date:" in data:
    Dates = data[12:31]
# делаем тоже самое, но уже ищем файлы с вирусной сигшнатурой и формируем аналогичную переменную, которая была в отдельном файле
for line in result:
   data = line.decode("utf-8")
   if "FOUND" in data:
       vali = data.split(': ')[0]
       list = list + "(" + "'" + vali + "'" + "," + "'" + Dates + "'" "), "
list = list + ')'
# к сожелению если дать переменную list типа char на вход для записи в БД, то будет выдана ошибка, поэтому мы конвертируем переменную list типа char в
# list типа tuple (кортеж) с которым спокойно работает метод executemany в psycapg2
# мы используем кортеж, а не список по двум причина, первая это кортеж быстрее, вторая это кортеж нельзя изменять в отличии от спика, что добавляет немного
# безопасности всему процессу передачи, немного конечно, но лучше так, чем вообще никак
list = literal_eval(list)


try:
    connection = psycopg2.connect(
        host=host,
        user=user,
        password=password,
        database=db_name
    )
    connection.autocommit = True
    with connection.cursor() as cursor:
        cursor.execute(
            "SELECT version();"
        )
        print(f"Server version: {cursor.fetchone()}")
    with connection.cursor() as cursor:
            cursor.executemany("INSERT INTO scan (folder, data) VALUES(%s, %s)", list)
            print("[INFO] Data was successfully inserted")
except Exception as _ex:
    print("[INFO] Error while working with PostgresSQL", _ex)
finally:
    if connection:
        connection.close()
        print("[INFO] PostgresSQL connection closed")
```

Это даже не винальная версия
