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
```

Метод  
```js
console.log(path.parse(__filename));
```  
возвращает объект в котором указывается корень диска, имя папки, имя файла, расширение файла, имя файла без расширения.

Метод  
```js
console.log(path.join(__dirname, 'test', 'second.html'));
```  
объединяет заданные сегменты пути вместе, используя в качестве разделителя разделитель данной конкретной платформы (для Linux - прямой слэш, для Windows - обратный слэш), результат - относительный путь  

Метод  
```js
console.log(path.resolve(__dirname, './test', '/second.html'));
```  
преобразует последовательность путей или сегментов пути в абсолютный путь и нормализует его: если в некоторых сегмента пути указываются слэши, в некоторых нет, всё равно будет сгенерирован правильный путь.