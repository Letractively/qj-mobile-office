# Настройка #

`vim /etc/mysql/my.cnf` [справка](http://dev.mysql.com/doc/refman/5.5/en/server-system-variables.html)

```
[mysqld]

max_connections  = 1000 # умолчание 151
wait_timeout     = 65   # умолчание 28800 сек
```

`mysql -p`

_вводим пароль..._

# Создание #

```
create database 'qq';
use qq;

set names utf8;

create table 'users' (
  'id' int(11) not null auto_increment,
  'sname' varchar(100),
  'bdate' datetime,
  'smoke' tinyint(1) default '0',
  'session_id' char(32),
  'about' varchar(250),

  primary key ('id'), key ('session_id')
  # индексы для быстрого поиска по id и session_id
)
default charset = utf8;

create table tasks (
  user_id int(11) not null,
  name varchar(20)
)
default charset = utf8;
```

# Добавление #

```
insert into users
(sname,bdate,session_id)
values
('Widenius','1962-03-03','9da1db7328af32d28b665af29706b874'); # формат даты YYYY-MM-DD HH:MM:SS

insert into tasks
(user_id,name)
values
('1','to create mysql');
```

# Изменение #

```
update users set about = 'the main author of the original version of the open-source MySQL database' where user_id = 1;
```

```
alter table users drop column smoke;
alter table users add column 'name' varchar(100);

alter table tasks change 'name' 'title' varchar(20);
alter table tasks modify 'title' varchar(100);

alter table users add index ('name','sname');
# индекс для быстрого поиска по составному условию, например:
# select * from users where name = 'Mega' and sname = 'Mind';
```

# Удаление #

```
delete from users where ...;
```

# Выборка #

```
select * from users where about is not null and id > 10;

select name,sname from users limit 5;     # первые 5
select name,sname from users limit 5,10;  # начиная с 6ого, 10 штук

select count(*) from tasks;               # подсчет количества
```

# Выборка с join #

```
select * from tA join tB on tA.x = tB.y join tC on tC.z = tB.z ... ;
```

```
select * from users cross join tasks; # декартово произведение
```

Пусть далее есть три пользователя `usr1`,`usr2`,`usr3`. У первого два задания `tx` и `ty`, у второго - одно `tz`, а третий без заданий.

## inner ##

```
select * from users u inner join tasks t on u.id = t.user_id;

# | u.id | u.sname | ... | t.user_id | t.title |
# +------+---------+-----+-----------+---------+
# | 1    | usr1    | ... | 1         | tx      |
# +------+---------+-----+-----------+---------+
# | 1    | usr1    | ... | 1         | ty      |
# +------+---------+-----+-----------+---------+
# | 2    | usr2    | ... | 2         | tz      |

select * from users u straight_join tasks t on u.id = t.user_id;

# результат тот же, но внешний цикл обязательно ведется по таблице users.
# если бы пользователей были бы тысячи, а задания были бы только у сотен
#  'users straight tasks' оказалось бы намного медленнее 'tasks straight users'
```

## outer ##

```
select * from users u left join tasks t on u.id = t.user_id;

# | u.id | u.sname | ... | t.user_id | t.title |
# +------+---------+-----+-----------+---------+
# | 1    | usr1    | ... | 1         | tx      |
# +------+---------+-----+-----------+---------+
# | 1    | usr1    | ... | 1         | ty      |
# +------+---------+-----+-----------+---------+
# | 2    | usr2    | ... | 2         | tz      |
# +------+---------+-----+-----------+---------+
# | 3    | usr3    | ... | null      | null    |

select * from tasks t right join users u on u.id = t.user_id;

# то же самое
```