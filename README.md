Домашнее задание к занятию «Резервное копирование баз данных» Макин Олег


Задание 1. Резервное копирование

Кейс
Финансовая компания решила увеличить надёжность работы баз данных и их резервного копирования.

Необходимо описать, какие варианты резервного копирования подходят в случаях:

1.1. Необходимо восстанавливать данные в полном объёме за предыдущий день.

1.2. Необходимо восстанавливать данные за час до предполагаемой поломки.

1.3.* Возможен ли кейс, когда при поломке базы происходило моментальное переключение на работающую или починенную базу данных.

Приведите ответ в свободной форме.

Задание 2. PostgreSQL

2.1. С помощью официальной документации приведите пример команды резервирования данных и восстановления БД (pgdump/pgrestore).

2.1.* Возможно ли автоматизировать этот процесс? Если да, то как?

Приведите ответ в свободной форме.

Задание 3. MySQL

3.1. С помощью официальной документации приведите пример команды инкрементного резервного копирования базы данных MySQL.

3.1.* В каких случаях использование реплики будет давать преимущество по сравнению с обычным резервным копированием?

Приведите ответ в свободной форме.



### Задание 1

1.1

Для восстановления данных в полном объёме за предыдущий день можно использовать дифференциальное резервное копирование. Этот метод создаёт копии файлов, которые были изменены с момента последней полной резервной копии, независимо от того, были ли они раньше включены в предыдущие дифференциальные копии. Для восстановления достаточно иметь одну полную копию и провести изменения относительно неё.

1.2

Для восстановления данных за час до предполагаемой поломки можно использовать инкрементное резервное копирование. В этом случае полная копия используется в качестве основы, которая дополняется только теми блоками данных, которые изменялись с момента проведения последнего бэкапа. Инкрементные копии создаются быстро, не занимают много места и минимально нагружают сеть. Для восстановления данных необходимо восстановить полные бэкапы на начало дня, а далее поочерёдно накатывать инкрементные бэкапы до искомого часа.

1.3.* 

Такой сценарий возможен и широко используется в системах, где важна высокая доступность данных и минимизация времени простоя. Этот процесс называется отказоустойчивостью. Кластер баз данных организуется из основной базы данных (master) и одной или несколько резервных (slave/replica). В случае отказа основной базы данных происходит автоматическое переключение на одну из реплик.


### Задание 2

2.1

Резервное копирование

pg_dump --host=localhost --port=5432 --username=postgres --format=c --file=backup_file.dump my_database

Где:
--host: имя хоста, на котором запущен сервер PostgreSQL (по умолчанию localhost).
--port: порт, на котором слушает сервер PostgreSQL (по умолчанию 5432).
--username: имя пользователя PostgreSQL.
--format=c: указывает, что резервная копия должна быть создана в формате custom (c), который поддерживает сжатие и удобен для последующего восстановления.
--file=backup_file.dump: путь и имя файла, куда будет сохранена резервная копия.
my_database: название базы данных, которую нужно сохранить.

Восстановление базы данных

pg_restore --host=localhost --port=5432 --username=postgres --dbname=new_database backup_file.dump

Где:
--host, --port, --username: параметры аналогичны команде pg_dump.
--dbname=new_database: название базы данных, в которую будет восстановлена резервная копия. Обратите внимание, что база данных должна уже существовать.
backup_file.dump: файл резервной копии, созданный командой pg_dump.

2.1.*

Автоматизация процесса резервного копирования


    #!/bin/bash

    # Настройки
    DATABASE_NAME="my_database"
    BACKUP_FILE="/path/to/backups/${DATABASE_NAME}_$(date +%Y-%m-%d_%H:%M:%S).sql"

    # Выполняем резервное копирование
    pg_dump --host=localhost --port=5432 --username=postgres --format=c --file="${BACKUP_FILE}" "${DATABASE_NAME}"
    
    # Проверяем успешность операции
    if [ $? -eq 0 ]; then
      echo "Backup of ${DATABASE_NAME} completed successfully!"
    else
      echo "An error occurred while backing up ${DATABASE_NAME}."
    fi

Запуск скрипта по расписанию с помощью cron


    0 0 * * * /path/to/scripts/backup.sh > /var/log/backup.log 2>&1


Автоматизация процесса восстановления

    #!/bin/bash

    # Настройки
    RESTORE_DATABASE_NAME="new_database"
    BACKUP_FILE="/path/to/backups/latest_backup.sql"

    # Выполняем восстановление
    pg_restore --host=localhost --port=5432 --username=postgres --dbname="${RESTORE_DATABASE_NAME}" "${BACKUP_FILE}"

    # Проверяем успешность операции
    if [ $? -eq 0 ]; then
      echo "Restore of ${RESTORE_DATABASE_NAME} completed successfully!"
    else
      echo "An error occurred while restoring ${RESTORE_DATABASE_NAME}."
    fi


### Задание 3

3.1

В этом примере используется опция --incremental-base=history:last_backup , с помощью которой mysqlbackup извлекает LSN последнего успешного (не TTS) полного или частичного резервного копирования из таблицы mysql.backup_history и выполняет инкрементное резервное копирование на его основе.

mysqlbackup --defaults-file=/home/dbadmin/my.cnf \
  --incremental --incremental-base=history:last_backup \
  --backup-dir=/home/dbadmin/temp_dir \
  --backup-image=incremental_image1.bi \
   backup-to-image


3.1.*

Использование реплики в MySQL может дать несколько преимуществ по сравнению с обычным резервным копированием в различных сценариях. Вот некоторые из них:

- Минимизация времени простоя: Репликация позволяет быстро переключиться на реплику в случае сбоя основной базы данных, что значительно сокращает время простоя по сравнению с восстановлением из резервной копии.
- Непрерывная доступность данных: Реплики могут быть использованы для обеспечения высокой доступности данных. Если основная база данных становится недоступной, реплика может взять на себя её функции.
- Нагрузочное распределение: Реплики могут использоваться для распределения нагрузки на чтение. Это позволяет разгрузить основную базу данных, улучшая производительность приложения.
- Геораспределенные системы: В случае геораспределенных систем реплики могут находиться в разных регионах, что позволяет улучшить доступность и скорость доступа к данным для пользователей из разных мест.
- Снижение нагрузки на резервное копирование: Использование реплик может снизить необходимость в частом резервном копировании, так как данные на репликах могут быть использованы для восстановления.
- Легкость в восстановлении: Восстановление данных с реплики может быть проще и быстрее, чем восстановление из резервной копии, особенно если резервные копии хранятся на внешних носителях.





















Инкрементное резервное копирование в MySQL осуществляется с помощью механизма бинарных журналов (binary logs). Эти журналы содержат информацию обо всех изменениях, внесённых в базу данных после определённого момента времени. Чтобы выполнить инкрементное резервное копирование, необходимо сначала создать полное резервное копирование, а затем периодически сохранять изменения, зафиксированные в бинарных журналах.
Шаги для настройки инкрементного резервного копирования

    Включение бинарного журналирования:Прежде всего, убедитесь, что ваш сервер MySQL настроен на ведение бинарных журналов. Откройте конфигурационный файл my.cnf (обычно располагается в /etc/mysql/my.cnf) и добавьте следующие строки в секцию [mysqld]:

log_bin = mysql-bin
binlog_format = ROW
server_id = 1

Параметр server_id должен быть уникальным для каждого сервера в вашей среде. После внесения изменений перезапустите сервер MySQL.
Создание полного резервного копирования:Перед началом инкрементного резервного копирования создайте полное резервное копирование базы данных. Это можно сделать с помощью инструмента mysqldump:

mysqldump -u root -p --all-databases --single-transaction --flush-logs --master-data=2 > full_backup.sql

Эта команда создаст дамп всех баз данных, включая метаданные мастера (позволяющие определить позицию в бинарном журнале на момент создания дампа).
Выполнение инкрементного резервного копирования:После создания полного резервного копирования вы можете начать инкрементное резервное копирование, сохраняя файлы бинарных журналов, созданные после полной копии. Для этого можно использовать команду mysqlbinlog:

mysqlbinlog --start-position=12345 mysql-bin.000001 > incremental_backup.sql

Здесь 12345 — это позиция в бинарном журнале, начиная с которой будут сохранены изменения. Эту позицию можно найти в файле полного резервного копирования, созданного ранее.
Применение инкрементного резервного копирования:Чтобы применить инкрементное резервное копирование, используйте команду mysql:

mysql -u root -p < incremental_backup.sql

Планирование регулярных инкрементных резервных копий:Вы можете настроить регулярные инкрементные резервные копии с помощью планировщика заданий, например, cron в Linux. Вот пример задания cron, которое выполняется каждый час:

0 * * * * mysqlbinlog --start-position=`cat /var/lib/mysql/binlog.index | tail -n 1 | awk '{print $1}'` /var/lib/mysql/mysql-bin.* > /path/to/incremental_backups/hourly_incremental_backup_`date +\%Y-\%m-\%d_\%H:\%M:\%S`.sql

Ротация файлов бинарных журналов:Важно регулярно очищать старые бинарные журналы, чтобы избежать переполнения диска. Для этого можно использовать параметр expire_logs_days в конфигурации MySQL:

expire_logs_days = 7

    Это установит срок хранения бинарных журналов в 7 дней.

Заключение

Инкрементное резервное копирование в MySQL требует тщательной настройки и регулярного обслуживания. Убедитесь, что ваши бинарные журналы правильно настроены и что у вас есть надежные процедуры ротации и архивации старых файлов журналов.




![img]()
![img]()
![img]()


