## Модуль PATH

Одним из стандартных модулей является path. Модуль path предназначен для того, чтобы работать с путями в Node.js. Можно получить имя файла, расширение файла, имя папки, указать путь к файлу.

Чтобы обратиться к стандартному модулю, его достаточно подключить:
```js
const path = require('path');
console.log(path)
```
Информацию о методах path можно найти на официальном сайте node.js https://nodejs.org/api/path.html

Рассмотрим некоторые из них:
```js
const path = require('path');
console.log(path.basename(__filename)) // index.js - имя файла
console.log(path.dirname(__filename)) // C:\Users\Admin\Desktop\nodejs-basic - название папки
console.log(path.extname(__filename)) // .js - расширение файла
console.log(path.parse(__filename)) // объект в котором указывается корень диска, имя папки, имя файла, расширение файла, имя файла без расширения
console.log(path.join(__dirname, 'test', 'second.html')) // объединяет заданные сегменты пути вместе, используя в качестве разделителя разделитель данной конкретной платформы, результат - относительный путь
console.log(path.resolve(__dirname, './test', '/second.html')) // ещё один способ генерировать пути, преобразует последовательность путей или сегментов пути в абсолютный путь
```