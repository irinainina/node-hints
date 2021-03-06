## Неблокирующий ввод/вывод

#### I/O (input/output)

I/O - input/output - означает ввод/вывод

- input - получение информации: от сетевых ресурсов, чтение с диска или файла, ввод с клавиатуры
- output - вывод информации: сохранение на диск, запись в файл, вывод в консоль

Это самые затратные по времени этапы работы программы. Сравните:
![](../node/images/io.png)

#### Блокирующий IO

Операции input/output происходят синхронно, одна за другой

```js
const fs = require('fs'); // подключаем модуль fs - file system
const data = fs.readFileSync('file.md'); // читаем файл
console.log(data); // выводим информацию в консоль
```

Это простой в реализации, но очень затратный по времени вариант: исполнение кода заблокировано, пока файл не будет полностью считан. Представьте, что программа читает "Войну и мир". Это долго, и всё это время пользователи не могут взаимодействовать с программой.  
Вторая проблема синхронного I/O - он не отказоустойчив. Если программа не сможет прочитать предложенный файл, например, если там окажутся неизвестные ей символы, программа остановит свою работу.

Синхронный или блокирующий I/O в Node.js используется очень редко:
  - если необходимо получить данные, без которых работа программы не может начаться. Например, информацию о настройках
  - может использоваться в консольных приложениях, у которых только один пользователь

#### Неблокирующий I/O

Неблокирующий I/O происходит асинхронно

```js
const fs = require('fs');
fs.readFile('file.md', (err, data) => {
  if (err) throw err;
  console.log(data);
});
```

В данном примере метод `fs.readFile()` является неблокирующим, и исполнение JavaScript может продолжаться, не дожидаясь окончания его работы. Когда файл прочитан, вызывается функция console.log().

Первым аргументом метода `fs.readFile()` является ошибка. Это принятый в Node.js паттерн, согласно которому первым параметром любой функции обратного вызова является объект ошибки. Если ошибка есть, асинхронная функция выбрасывает ошибку и прекращает свою работу. Если ошибки нет, функция считывает данные и выводит их в консоль.

#### Опасность смешивания блокирующего и неблокирующего кода

При работе с I/O следует избегать смешивания блокирующего и неблокирующего кода . Взглянём на пример:

```js
const fs = require('fs');
fs.readFile('file.md', (err, data) => {
  if (err) throw err;
  console.log(data);
});
fs.unlinkSync('file.md');
```

В этом примере метод `fs.unlinkSync()` с высокой вероятностью будет выполнен до `fs.readFile()`. Это приведёт к удалению файла до его прочтения. Лучше переписать этот код в неблокирующем виде, что гарантирует правильный порядок исполнения методов:

```js
const fs = require('fs');
fs.readFile('file.md', (readFileErr, data) => {
  if (readFileErr) throw readFileErr;
  console.log(data);
  fs.unlink('file.md', (unlinkErr) => {
    if (unlinkErr) throw unlinkErr;
  });
});
```

В этом примере **неблокирующий вызов** метода `fs.unlink()` расположен внутри функции обратного вызова `fs.readFile()`. Такой подход гарантирует правильную последовательность операций.

В основе работы Node.js лежат **неблокирующий ввод/вывод** и **асинхронность**. Благодаря этому приложения на Node.js работают намного быстрее и могут обслуживать намного больше клиентских запросов в единицу времени, чем аналогичные приложения, разработанные на большинстве других языков программирования.