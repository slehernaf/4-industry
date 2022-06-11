# Отчет

## Разбор работы антивируса ClamAV
Clam AntiVirus - пакет антивирусного ПО, работающий во многих операционных системах, включая Unix-подобные ОС, OpenVMS, Microsoft Windows и Apple Mac OS X.
Выпускается под GNU General Public License и является свободным программным обеспечением.
Главная цель Clam AntiVirus - интеграция с серверами электронной почты для проверки файлов, прикрепленных к сообщениям. В пакет входит масштабируемый многопоточный демон clamd, управляемый из командной строки сканер clamscan, а также модуль обновления сигнатур по Интернету freshclam.
Хоть основная функция ClamAV - это проверка файлов прикрепленных к почтовым сообщениям, он также может сканировать и систему в целом, как в ручном так и в автоматическом режиме, все зависит от конфигураций. 
В дополнение к самому ClamAV можно установить ClamTK, который просто является визуальной оболочкой для ClamAV, чтобы не использовать его через консоль, мы этого не делали, так как весь функционал раскрывается непосредственно при использовании консольной версии. Для своих целей мы использовали ClamAV в качестве антивируса, а не сканера почтового трафика на шлюзах.
Стоит немного оговориться, хоть ClamAV и является открытым продуктом, но он все же принадлежит Cisco Systems, из-за чего могут быть проблемы с обновлением баз данных.
Установка и первоначальная настройка ClamAV для Linux максимально проста.
Первым делом обновляем систему: ```apt-get update```
После чего устанавливаем сам антивирус и демона для того чтобы ClamAV смог взаимодействовать с файлами: ```apt-get install clamav clamav-daemon```
После чего переходим в папку с конфигами при помощи команды: ```open /usr/local/etc/```
Видим два файла clamd.conf.sample и freshclam.conf.sample, стираем в названии обоих “.sample”, открываем каждый конфиг удаляем вверху слово “Example”, и вставляем строку ```DatabaseDirectory /var/lib/clamav```, тут указан путь к базам данных в которых содержатся вирусные сигнатуры (данные базы обновляются автоматически при помощи сервиса clamav-freshclam), для автоматического обновления баз данных необходимо еще в файле freshclam.conf добавить строку ```PrivateMirror clmvupd.deltamoby.ru```, это зеркало с базами вирусных сигнатур для ClamAV, доступ к которым имеют все.
Для сканирования системы первым делом необходимо запустить антивирус делается это командой: ```sudo service clamav-daemon start```, после чего проверяем запустился ли антивирус командой ```sudo service clamav-daemon status```, для удобства пропишем ```sudo systemctl enable clamav-daemon.service```, чтобы данная служба запускалась автоматически при включении рабочей станции.

Теперь нам необходимо что-то просканировать, но для начала давайте создадим файл с вирусной сигнатурой в нашей системе, для этого мы использовали известный тестовыйы файл elicar. После того как мы создали то, что необходимо будет найти антивирусу, нам нужно понять, а как искать. Для этого все также используется консоль в которую прописывается clamscan опции /путь к файлу/, перечень опций не столь обширен:

* -V, --version - вывести версию программы;
* -v, --verbose - максимально подробный вывод;
* -a, --archive-verbose - выводить сообщения о проверяемых файлах внутри архивов;
* --debug - отображать отладочные сообщения;
* --quiet - выводить минимум информации;
* --no-summary - не отображать результат сканирования;
* -i, --infected - выводить информацию только об инфицированных файлах;
* --bell - воспроизводить звуковой сигнал при обнаружении вируса;
* -d, --database - загрузить вирусную базу данных из файла;
* -l, --log - сохранить результат сканирования в файл;
* -r, --recursive - рекурсивное сканирование директорий и поддиректорий;
* --move - перемещать инфицированные файлы в указанную папку;
* --copy - копировать инфицированные файлы в указанную папку;
* --exclude, --exclude-dir - не сканировать файлы и папки, имена которых подпадают под шаблон;
* --remove - удалять все инфицированные файлы;
* --scan-pe=yes/no - сканировать исполняемые файлы Windows, по умолчанию включено;
* --scan-elf=yes/no - сканировать исполняемые файлы Linux, по умолчанию включено;
* --scan-ole2=yes/no - сканировать документы Office, по умолчанию включено;
* --scan-archive=yes/no - сканировать архивы, по умолчанию включено;
* --max-filesize - максимальный размер данных, извлекаемых из архива, по умолчанию 25 Мб;
* --max-files - извлекать не больше указанного количества файлов из сканируемых файлов (архивы, документы, любой вид контейнеров), по умолчанию 10000;
* --max-recursion - максимальная глубина сканирования папок в архиве, по умолчанию 16;
* --max-dir-recursion - максимальная глубина сканирования папок в файловой системе, по умолчанию 15.

Хорошо мы теперь полностью готовы к работе, мы хотим чтобы ClamAV, выводил данные только при обнаружении файлов с вирусными сигнатурами, находящимся “/home/kal/Загрузки/”, после чего сохранял отчет о сканировании в файл под названием “scan.log” в домашнюю дерикторию, по умолчанию это /home/"имя пользователя", для этого нам необходимо сформировать команду, которая будет иметь следующий вид: 
```clamscan -i /home/kal/Загрузки/ -l scan.log / &```
Здесь, “/ &” используется для удобства, чтобы выполнение команды происходило в фоновом режиме.

Видим что в домашней директории был создан файл “scan.log”, его содержимое совпадает с тем, что выводилось на терминале, а именно:
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
Time: 139.733 sec (2 m 19 s)
Start Date: 2022:05:13 22:01:10
End Date:   2022:05:13 22:03:30
```

Также давайте немного разберем базы данных вирусных сигнатур, которые использует ClamAV. Базы данных хранятся в директории указанных в конфигурационном файле, а именно “/var/lib/clamav”, всего есть три файла 	“bytecode.cvd”, “daily.cvd”, “main.cvd”. “bytecode.cvd” - содержит оперативные изменения, а именно дополнительные методы обнаружения вирусов и улучшения для антивируса ClamAV. “daily.cvd” - база данных вирусных сигнатур, которая ежедневно обновляется.“main.cvd” - основная база данных вирусных сигнатур.
Формат .cvd использует это программа и некоторые другие специализированные, чтобы просмотреть содержимое баз данных нужно ввести команду: ```sudo sigtool -u FileName```
В нашем случае мы скопировали файл “daily.cvd” в папку “clamavDB” на рабочем столе, поэтому нам нужно первым делом из консоли добраться до данной папки путем простой команды: ```cd Рабочий\ стол/clamavD```B и ввести команду выше, а именно ```sudo sigtool -u daily.cvd``` . Тем самым мы получаем все файлы хранящиеся в “daily.cvd” в нашей папке “clamavDB”.
Нас интересует файл “daily.mdb”, без специализированных программ мы не сможем прочитать интересующий нас файл, но мы можем конвертировать “daily.mdb” в “daily.xlsx” и уже его прочитать как файл офисной таблицы. Тем самым мы можем узнать какая сигнатура какому типу вирусов принадлежит.

## Разбор системы управления баз данных PostgreSQL 

PostgreSQL - свободная объектно-реляционная система управления базами данных (СУБД).
Существует в реализациях для множества UNIX-подобных платформ, включая AIX, различные BSD-системы, HP-UX, IRIX, Linux, macOS, Solaris/OpenSolaris, Tru64, QNX, а также для Microsoft Windows.
PostgreSQL базируется на языке SQL и поддерживает многие из возможностей стандарта SQL:2011. В PostgreSQL версии 12 есть следующие ограничения:

|   |  |
| ------------- | ------------- |
|Максимальный размер базы данных|Нет ограничений|
|Максимальный размер таблицы|32 Тбайт|
|Максимальный размер поля|1 Гбайт|
|Максимум записей в таблице|Ограничено размерами таблицы|
|Максимум полей в записи|250—1600, в зависимости от типов полей|
|Максимум индексов в таблице|Нет ограничений|

Сильными сторонами PostgreSQL считаются:

* высокопроизводительные и надёжные механизмы транзакций и репликации;
* расширяемая система встроенных языков программирования: в стандартной поставке поддерживаются PL/pgSQL, PL/Perl, PL/Python и PL/Tcl; дополнительно можно использовать PL/Java, PL/PHP, PL/Py, PL/R, PL/Ruby, PL/Scheme, PL/sh и PL/V8, а также имеется поддержка загрузки модулей расширения на языке C[10];
* наследование;
* возможность индексирования геометрических (в частности, географических) объектов и наличие базирующегося на ней расширения PostGIS;
* встроенная поддержка слабоструктурированных данных в формате JSON с возможностью их индексации;
* расширяемость (возможность создавать новые типы данных, типы индексов, языки программирования, модули расширения, подключать любые внешние источники данных).

### Установка и использование PostgreSQL
Для начала установим PostgreSQL, для этого первым делом проверим наличие обновлений и обновимся в случае необходимости ```sudo apt update```, после чего устанавливаем PostgreSQL, благо его установка приметивная и не требует "танцев с бубном", как с ClamAV, нам просто нужно прописать ```apt-get install postgresql-12```, также для удобсва включаемавтозагрузку СУБД, прописав ```sudo systemctl enable postgresql``` и начинаем рабоать. Первым делом необходимо зайти для этого пропишем в консоли ```sudo -i -u postgres```, хорошо мы открыли PostgreSQL, дальше нам необходимо зайти прописав ```psql```, теперь мы зашли в PostgreSQL, под учетной записью администратора, сейчас мы не будем разбирать всех возможностей данного СУБД и учиться им пользоваться, мы изучим базовые понятия и немного поизучаем SQL язык, для этого первым делом изменим пароль для учетной записи, нам нужно зайдя из-под администратора прописать ```ALTER USER postgres PASSWORD 'password';```, где в одинарных кавычках указывактся пароль, мы прописали "clamav", далее создадим базу данных ```createdb 'db_name';``` или ``` CREATE DATABASE 'db_name';```, где в одинарных кавычках указыкавется название базы данных, в нашем случае так как мы будем в БД записывать вирусы,то и назовем соответствующе "virus". Проверяем нашу БД, а точнее смотрим какиее вообще есть БД в нашей системе ```\l```. Если все хорошо и БД есть, ее владелец "postgres", то мы выходим из под учетной записи администратора```\q``` и заходим в ранее созданную БД ```psql -d virus```. БД пуста, создадим в ней таблицу ```CREATE TABLE scan(id serial PRIMARY KEY, folder varchar(8000), data char(20));``` Тут мы создали таблицу под названием "scan" с тремя столбцами:
* id
* folder
* data

и сказали какими они будут. Так _id_ будет автоматически создаваться для каждой новой записи, _folder_ может записать в себя до 8000 символов(ограничение), а главное отличие в том что если в записи будут использованы не все символы, то ничего страшного их просто не будет, в отличии от третьего столбца _data_, в котором строго 20 символов, если мы введем больше, то будет ошибка, если введем меньше, то незаполненное пространство символов заполнятся автоматически пустотой, что не есть хорошо.
Проверим как поживает наша сделав запрос на просмотр ее содержимого ```Select*from 'table_name';``` где в одинарных кавычках название таблицы в базе данных из под которой мы сидим, в нашем случае пропишем ```Select*from scan;``` . Если все хорошо то мы увидим три пустых столбца, нажимаем ```q``` чтобы выйти из просмотра и займемся наполнением таблицы в автоматическом режиме используя python.

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

Это даже не винальная версия, как вы могли заметить мы два раза перебираем выходные данные, чтобы найти дату и время, после чего ищем все найденные файлы с вирусной сигнатурой и формирвем переменную, нам не особо понравилось что приходится дважды проходиться по данным, поэтому мы решили что нам нужно самим формировать переменную Dates, чтобы избавиться от лишнего прохода по файлу, это мы и реализовали в финальной версии на данный момент "своеобразного коннектора" для ClamAV и PostgreSQL выглядит так:
```
import subprocess
import psycopg2
import datetime
from config import host, user, password, db_name
from ast import literal_eval

list = '('
print('input the path to the folder or file')
cmd = "clamdscan -i " + input()
process = subprocess.Popen(cmd,shell=True,stdin=None,stdout=subprocess.PIPE,stderr=subprocess.PIPE)
result = process.stdout.readlines()
error = process.stderr.readlines()
if len(error)!= 0:
    print(error)
    exit()
# берем значение даты с машины, мы берем, год, месяц, день, час, минуты, секунду и милисекунду на момент выполнения запроса и записываем в переменную Dates
Dates = datetime.datetime.now()
# Переводим переменную Dates в переменную типа str форматируя ее таким образом чтобы выводился "год:месяц:день час:минута:секунда" 
Dates = Dates.strftime("%Y:%m:%d %H:%M:%S")

for line in result:
   data = line.decode("utf-8")
   if "FOUND" in data:
       vali = data.split(': ')[0]
       list = list + "(" + "'" + vali + "'" + "," + "'" + Dates + "'" "), "
list = list + ')'
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


```
from tkinter import filedialog
from tkinter import *
import tkinter.messagebox as mb
import subprocess
import psycopg2
import datetime
from ast import literal_eval


host = "127.0.0.1"
user = "postgres"
password = "clamav"
db_name = "virus"
list= ''
sc_browse = ''
log_browse = ''
infested_browse = ''
ws = Tk()
ws.title('ClamAv connector app')
ws.geometry('900x280')

def log_browse():
    global log_browse
    log_browse = filedialog.askdirectory()

def scan_browse():
    global sc_browse
    if rbt.get() == 0:
        sc_browse = filedialog.askdirectory()
    elif rbt.get() == 1:
        sc_browse = filedialog.askopenfilename()

def isChecked():
    global log_browse, infested_browse

    if cb0.get() == 1:
        btn0['state'] = NORMAL
        btn0.configure(text='Путь логов')
    elif cb0.get() == 0:
        btn0['state'] = DISABLED
        btn0.configure(text='Выберете "Сохранить лог файл"')

    if cb1.get() == 1:
        cbt2['state']= DISABLED
    elif cb1.get() == 0:
        cbt2['state'] = NORMAL

    if cb2.get() == 1:
        btn2['state'] = NORMAL
        btn2.configure(text='Путь папки, для перемещения инфицированных файлов')
        cbt1['state']= DISABLED
    elif cb2.get() == 0:
        btn2['state'] = DISABLED
        btn2.configure(text='Выберете "Перемещать инфицированные файлы в указанную папку"')
        cbt1['state'] = NORMAL

def infest_browse():
    global infested_browse
    infested_browse= filedialog.askdirectory()

def scan():
    global list, sc_browse, log_browse, infested_browse
    if type(sc_browse) != str or sc_browse=='':
        mb.showerror("Ошибка", "Не выбран путь сканирования")
        return
    msg = 'Будет сканироваться:'+sc_browse+'\n'
    if cb0.get() ==1:
        if type(log_browse) != str or log_browse == '' :
            mb.showerror("Ошибка", "Не выбран путь сохранения логов сканирования")
            return
        msg = msg + 'Логи будут сохранены в:' + log_browse + '/scan.log' + '\n'
    if cb2.get() == 1:
        if type(infested_browse) != str or infested_browse == '':
            mb.showerror("Ошибка", "Не выбран путь для перемещения инфицированных файлов")
            return
        msg = msg + 'Вирусные файлы будут перемещены в:'+ infested_browse+ '\n'
    if cb1.get() == 1:
        msg = msg + 'Найденные вирусы будут удалены'+'\n'
    mb.showinfo("Информация", msg)
    quest = mb.askyesno("ClamAV Scanning", "Вы уверены что хотите сканировать с данными параметрами?")
    if quest == False:
        return
    else:
        list = '('
        cmd = "clamdscan -i " + sc_browse.replace(' ', '\ ')
        if cb2.get() == 1:
            cmd = cmd + ' --move ' + infested_browse.replace(' ', '\ ')
        if cb1.get() == 1:
            cmd = cmd + ' --remove'
        if cb0.get() == 1:
            cmd = cmd + ' -l ' + log_browse.replace(' ', '\ ') + '/scan.log'
        print(cmd)
        process = subprocess.Popen(cmd, shell=True, stdin=None, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
        result = process.stdout.readlines()
        error = process.stderr.readlines()
        if len(error) != 0:
            print(error)
            mb.showerror("Ошибка", error)
            return
        mb.showinfo("Информация", "Пожалуйста подождите, идет сканирование")
        Dates = datetime.datetime.now()
        Dates = Dates.strftime("%Y:%m:%d %H:%M:%S")
        ss =''
        for line in result:
            data = line.decode("utf-8")
            if "FOUND" in data:
                vali = data.split(': ')[0]
                ss = ss + vali + "\n"
                list = list + "(" + "'" + vali + "'" + "," + "'" + Dates + "'" "), "
        list = list + ')'
        list = literal_eval(list)
        print(list)
        mb.showwarning("Найденные вирусы",ss)
        psql = mb.askyesno("PostgreSQL","Записать найденные вирусы в базу данных?")
        if psql == False:
            return
        else:
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
                    mb.showinfo("Информация","Data was successfully inserted")
            except Exception as _ex:
                print("[INFO] Error while working with PostgresSQL", _ex)
                mb.showerror("Error while working with PostgresSQL", _ex)
            finally:
                if connection:
                    connection.close()
                    print("[INFO] PostgresSQL connection closed")
                    mb.showinfo("Информация", "PostgresSQL connection closed")
                    
cb0 = IntVar()
cb1 = IntVar()
cb2 = IntVar()
rbt = IntVar()

cbt0 = Checkbutton(ws, text="Сохранить лог файл?", variable=cb0, onvalue=1, offvalue=0, command=isChecked)
cbt1 = Checkbutton(ws, text="Удалять все инфицированные файлы?", variable=cb1, onvalue=1, offvalue=0, command=isChecked)
cbt2 = Checkbutton(ws, text="Перемещать инфицированные файлы в указанную папку?", variable=cb2, onvalue=1, offvalue=0, command=isChecked)

btn0 = Button(ws, text='Выберете "Сохранить лог файл"', state=DISABLED, command=log_browse)
btn1 = Button(ws, text='Выберете путь', command=scan_browse)
btn2 = Button(ws, text='Выберете "Перемещать инфицированные файлы в указанную папку"', state=DISABLED, command=infest_browse)
btn3 = Button(ws, text='Scaning', command=scan)

rbt0 = Radiobutton(text="Папку с вложенными папками", value=0, variable=rbt)
rbt1 = Radiobutton(text="Один файл", value=1, variable=rbt)

cbt0.pack(side=RIGHT)
cbt1.pack()
cbt2.pack()

rbt0.pack()
rbt1.pack()

btn0.pack()
btn1.pack()
btn2.pack()
btn3.pack()

ws.mainloop()
```
