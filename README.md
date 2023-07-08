# Пример развертывания DocHub

**Цели примера:**
1. Показать как можно разделить DocHub на различные репозитории в зависимости от их назначения
2. Показать как реализовать контеризацию выделенных репозиториев на базе Docker
3. Показать как можно оптимизировать работу с контейнерами DocHub используя docker compose


# Авторские права
1. Вся работа по настройке контейнеров Docker была выполнена Александром Трубниковым https://t.me/cu3blukekc
2. Пример репозитория с моделью DocHub и текущая инструкция были сделаны Валентином Козловым https://t.me/i_frog_i. Базовый пример модели был взят [отсюда](https://github.com/rpiontik/DocHubExamples/tree/main/src/repository_structure_example).
3. Репозиторий для сборки сервера PlantUML принадлежит Владиславу Маркину https://t.me/vlad_markin и был взят [отсюда](https://github.com/vlad-markin/plantuml-server/tree/dochub-v2).

# Суть примера
В процессе работы с DocHub стало понятно, что процесс наполнения архитектурного озера данных и процесс реализации и доработки метамодели - это разные процессы.

Помимо того что в общем случае этим занимаются разные люди, так еще оказалось, что первый процесс требует оперативной скорости обновления, в второй качества реализации.

После того как Рома дал возможность разделить метамодель DocHub и данные было приято решение переструктурировать подход управления DocHub.

Что было сделано:
1. Мы выделили в отельный репозиторий метамодель (metamodel), при этом дефолтная метамодель была кастомизирована. Для этого репозитория был реализован полноценный пайплан разработки со всеми этапами процесса включая полноценное тестирование. Здесь важно, что матамодель раздаётся через nginx.
2. Мы выделили в отельный репозиторий данные (manifest). Для этого репозитория был реализован упрощенный пайплан, который включает запуск штатных валидаторов DocHub. Если автовалидация проходит успешно, то данные сразу катятся в прод. Здесь важно, что данные архитектурного озера раздаются через nginx.
3. Помимо разделения мамамодели и данных, мы выделили репозиторий с бэкендом (backend). Внутри этого репозитория живут кастомные плагины на vue.js, поэтому в настройках этого репозитория учитывается необходимость автоматической установки зависимостей в случае добавления новых плагинов.
4. И четвертый репозиторий, который мы выделили стал фронтенд (frontend). В этом репозитории задаются базовые переменные DocHub.

Есть еще желание выделить плагины в отдельный репозиторий но мы пока не поняли как это сделать.

Такой подход позволяет нам:
1. Управлять разными процесса по разному
2. Существенно ускорить процесс выкатки изменений, причем как данных так и метамодели
3. Упростить выкатку изменений для архитекторов отвечающих за данные
4. Улучшить качество метамодели, так как сейчас у нас есть время все протестировать


## Файловая структура примера
* **backend** - в этой папке хранятся все данные, связанные бэкендом.
    * **dochub** - так как у нас наши плагины живут здесь, то мы используем свой fork. Подключаем мы его через `git submodule add`. В примере оригинальный DocHub.
    * **Dockerfile** - настройка контейнера Docker.    
    * **entrypoint.sh** - запуск бэкенда.
* **frontend** - в этой папке хранятся все данные, связанные фронтендом.
    * **dochub** - так как у нас есть наши изменения в ядре DocHub, то мы используем свой fork. Подключаем мы его через `git submodule add`. В примере оригинальный DocHub.
    * **Dockerfile** - настройка контейнера Docker.
    * **entrypoint.sh** - получение переменных DocHub и запуск фронтенда.
    * **nginx.conf** - настройка nginx.
* **manifest** - в этой папке хранятся все данные архитектурного озера
    * **manifest** - здесь мы подключаем различные репозитории с данными. Это и уровень L1 и уровень L2. Все репозитории мы подключаем как сабмодули, но тут нужно учитывать, что в этом случае при обновлении нужно будет менять ветку на дефолтную иначе обновления нне будет, так как при создании сабмодуля фиксируется конкретный коммит. У нас для автоматизации был написан специальный скрипт, который пробегается по всем подключенным репозиториям, меняет ветку и запускает git pull.
    * **Dockerfile** - настройка контейнера Docker.    
    * **nginx.conf** - настройка nginx через который раздаются данные для бэкенда.
* **metamodel** - в этой папке хранятся как дефолтная метамодель, так и наша.
    * **metamodel** - здесь у нас хранятся все метамодели, а также вспомогательные элементы.
        * **datasets** - здесь у нас хранятся все dtasets.
        * **dochub** - здесь у нас хранится дефолтная метамодель, а также дефолтные инструменты расширения этой метамодели.
        * **jsonata** - здесь у нас хранится код jsonata, который мы переиспользуем через eval.
        * **swamp** - здесь у нас хранится наша метамодель.        
    * **Dockerfile** - настройка контейнера Docker.    
    * **nginx.conf** - настройка nginx через который раздаются данные для бэкенда.
* **docker-compose.yaml** - пакетный запуск контейнеров Docker.

## Использование
### Жаркий старт
1. Скачайте себе пример
2. Зайдите в корень репозитория и выполните команду docker-compose up или docker compose up (v2).
3. Откройте браузер и наберите http://localhost:8080/ 
4. Успех!
5. Празднование успеха!

### Если нужно, что-то поменять на горячую
Для того чтоыб передернуть бэкенд Рома сделал secret для доступа к перезагрузке данных архитектуры:
VUE_APP_DOCHUB_RELOAD_SECRET=9H...FR

1. Создайте в корне репозитория файл .env
2. Внесите в файл .env значение переменной VUE_APP_DOCHUB_RELOAD_SECRET=9H...FR
3. Перезапустите контейнеры если они были запущены
4. Внесите изменения в озеро данных или в метамодель
5. Выполните скрипт reload_backend.sh
6. Проверьте что новые изменения подтянулись на портал DocHub

