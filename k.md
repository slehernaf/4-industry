## Работа с python

Первым делом установим python выполнив ряд команд ```sudo apt update```, как обычно проверили и обновились. ```sudo apt install build-essential zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev libssl-dev libsqlite3-dev libreadline-dev libffi-dev curl libbz2-dev```, установили зависимости, ```wget https://www.python.org/ftp/python/3.9.1/Python-3.9.10.tgz```, скачали исходник, ```tar -xf Python-3.9.10.tgz```, распаковали архив с исходником, ```cd Python-3.9.10```, зашли в распакованный архив и запусчкаем оптимизацию двоичного кода командой ```./configure --enable-optimizations```, запускаем процесс сборкии ```make -j 2```. Теперь же мы устанавливаем сами двоичные файлы ```sudo make altinstall``` и проверяем версию ```python3.9 --version```. Для удобства мы также установили _pycharm_ простой командой ```sudo snap install pycharm-community --classic```.

### Первый опыт 






### Второй опыт
