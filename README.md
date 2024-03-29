# Домашнее задание к занятию «Языки разметки JSON и YAML»

### Цель задания

В результате выполнения задания вы:

* познакомитесь с синтаксисами JSON и YAML;
* узнаете, как преобразовать один формат в другой при помощи пары строк.

### Чеклист готовности к домашнему заданию

1. Установлена библиотека PyYAML для Python 3.

### Инструкция к заданию 

1. Скопируйте в свой .md-файл содержимое этого файла, исходники можно посмотреть [здесь](https://raw.githubusercontent.com/netology-code/sysadm-homeworks/devsys10/04-script-03-yaml/README.md).
3. Заполните недостающие части документа решением задач — заменяйте `???`, остальное в шаблоне не меняйте, чтобы не сломать форматирование текста, подсветку синтаксиса. Вместо логов можно вставить скриншоты по желанию.
4. Любые вопросы по выполнению заданий задавайте в чате учебной группы или в разделе «Вопросы по заданию» в личном кабинете.

### Дополнительные материалы

1. [Полезные ссылки для модуля «Скриптовые языки и языки разметки».](https://github.com/netology-code/sysadm-homeworks/tree/devsys10/04-script-03-yaml/additional-info)

------

## Задание 1

Мы выгрузили JSON, который получили через API-запрос к нашему сервису:

```
    { "info" : "Sample JSON output from our service\t",
        "elements" :[
            { "name" : "first",
            "type" : "server",
            "ip" : 7175 
            }
            { "name" : "second",
            "type" : "proxy",
            "ip : 71.78.22.43
            }
        ]
    }
```
  Нужно найти и исправить все ошибки, которые допускает наш сервис.

### Ваш скрипт:

```
   {
        "info": "Sample JSON output from our service\t",
        "elements": [
            {
                "name": "first",
                "type": "server",
                "ip": 7175
            },
            {
                "name": "second",
                "type": "proxy",
                "ip": "71.78.22.43"
            }
        ]
    }
```

---

## Задание 2

В прошлый рабочий день мы создавали скрипт, позволяющий опрашивать веб-сервисы и получать их IP. К уже реализованному функционалу нам нужно добавить возможность записи JSON и YAML-файлов, описывающих наши сервисы. 

Формат записи JSON по одному сервису: `{ "имя сервиса" : "его IP"}`. 

Формат записи YAML по одному сервису: `- имя сервиса: его IP`. 

Если в момент исполнения скрипта меняется IP у сервиса — он должен так же поменяться в YAML и JSON-файле.

### Ваш скрипт:

```
#!/usr/bin/env python3

import socket
import json
import yaml


file_old = open('ip_add.log', 'r')
stage_ipadds = json.loads(file_old.read())

ip_item = []
dns_item = ["drive.google.com", "mail.google.com", "google.com"]
for resolv in dns_item:
    ip_item.append(socket.gethostbyname(resolv))
current_ipadds = dict(zip(dns_item, ip_item))

stage_ipadds_new = open('ip_add.log', 'w')
stage_ipadds_new.write(json.dumps(current_ipadds))
stage_ipadds_new.close()

print('For check:')
print(current_ipadds)
print(f'{stage_ipadds}\n')

with open('first.json', 'w') as js:
    js.write('')
with open('second.yml', 'w') as ym:
    ym.write('')

for i in current_ipadds:
    if (current_ipadds[i] == stage_ipadds[i]):
        print(f'<{i}> - <{current_ipadds[i]}>')
        rr = {i: current_ipadds[i]}
        with open('first.json', 'a') as js:
            js.write(f'{json.dumps(rr)}\n')
        with open('second.yml', 'a') as ym:
            ym.write(yaml.dump(rr))
    else:
        print(f'[ERROR] <{i}> IP mismatch: <{stage_ipadds[i]}> <{current_ipadds[i]}>')
        rr2 = {i: current_ipadds[i]}
        with open('first.json', 'a') as js:
            js.write(f'{json.dumps(rr2)}\n')
        with open('second.yml', 'a') as ym:
            ym.write(yaml.dump(rr2))
```

### Вывод скрипта при запуске во время тестирования:


![111](https://user-images.githubusercontent.com/123774335/233934987-9f103b50-a2bc-49a0-8706-7f1dcff92314.png)



### JSON-файл(ы), который(е) записал ваш скрипт:

```
{"drive.google.com": "142.251.1.194"}
{"mail.google.com": "173.194.222.83"}
{"google.com": "173.194.222.101"}
```

### YAML-файл(ы), который(е) записал ваш скрипт:

```
drive.google.com: 142.251.1.194
mail.google.com: 173.194.222.83
google.com: 173.194.222.101
```

---

## Задание со звёздочкой* 

Это самостоятельное задание, его выполнение необязательно.
____

Так как команды в нашей компании никак не могут прийти к единому мнению о том, какой формат разметки данных использовать: JSON или YAML, нам нужно реализовать парсер из одного формата в другой. Он должен уметь:

   * принимать на вход имя файла;
   * проверять формат исходного файла. Если файл не JSON или YAML — скрипт должен остановить свою работу;
   * распознавать, какой формат данных в файле. Считается, что файлы *.json и *.yml могут быть перепутаны;
   * перекодировать данные из исходного формата во второй доступный —  из JSON в YAML, из YAML в JSON;
   * при обнаружении ошибки в исходном файле указать в стандартном выводе строку с ошибкой синтаксиса и её номер;
   * полученный файл должен иметь имя исходного файла, разница в наименовании обеспечивается разницей расширения файлов.

### Ваш скрипт:

```python
import json
import yaml
import sys
import os

def parse_file(filename):
    # Проверяем расширение файла
    if not filename.endswith('.json') and not filename.endswith('.yml'):
        print('Error: Invalid file extension')
        sys.exit(1)

    # Определяем формат данных в файле
    try:
        with open(filename) as f:
            data = json.load(f)
            format = 'json'
    except ValueError:
        try:
            with open(filename) as f:
                data = yaml.load(f, Loader=yaml.FullLoader)
                format = 'yaml'
        except yaml.YAMLError as e:
            print('Error:', e)
            sys.exit(1)

    # Преобразуем данные в другой формат
    if format == 'json':
        converted_data = yaml.dump(data)
        new_filename = os.path.splitext(filename)[0] + '.yml'
    else:
        converted_data = json.dumps(data, indent=4)
        new_filename = os.path.splitext(filename)[0] + '.json'

    # Записываем преобразованные данные в новый файл
    with open(new_filename, 'w') as f:
        f.write(converted_data)

if __name__ == '__main__':
    if len(sys.argv) < 2:
        print('Usage: python parser.py <filename>')
        sys.exit(1)

    filename = sys.argv[1]
    parse_file(filename)
```

Пример использования:

```bash
$ python parser.py data.json
$ python parser.py data.yml
```

### Правила приёма домашнего задания

В личном кабинете отправлена ссылка на .md-файл в вашем репозитории.

-----

### Критерии оценки

Зачёт:

* выполнены все задания;
* ответы даны в развёрнутой форме;
* приложены соответствующие скриншоты и файлы проекта;
* в выполненных заданиях нет противоречий и нарушения логики.

На доработку:

* задание выполнено частично или не выполнено вообще;
* в логике выполнения заданий есть противоречия и существенные недостатки.  
 
Обязательными являются задачи без звёздочки. Их выполнение необходимо для получения зачёта и диплома о профессиональной переподготовке.

Задачи со звёздочкой (*) являются дополнительными или задачами повышенной сложности. Они необязательные, но их выполнение поможет лучше разобраться в теме.
