# Рефакторинг кода приложения для заметок

В прошлый раз мы написали [консольное приложение для работы с заметками](node/projects/notes.md).
Наше приложение работает отлично, но имеет некоторые недостатки:

1. ###### Устаревший подход к написанию асинхронного кода:
   На данный момент в нашем приложении используется подход основанный на [функциях обратного вызова(callback)](https://developer.mozilla.org/ru/docs/Glossary/Callback_function).
   Конечно, данный подход всё ещё работает, но является не предпочтительным в силу того, что неминуемо ведёт ваш код в так называемый [callback hell](http://callbackhell.com/).
   В современном javascript есть инструменты, такие как [promise](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Global_Objects/Promise) и [async\await](https://developer.mozilla.org/ru/docs/Learn/JavaScript/Asynchronous/Async_await), способные исправить эту ситуацию и мы попробуем применить их на практике.
2. ###### Структура приложения:
   На данный момент наше приложение является простым скриптом, который исполняется сверху вниз, но давайте разберёмся почему это может быть плохо.
   Представьте себе, что мы решили добавить функционал для вывода заметок с определённым набором символов(будь то дата или слово). Всего несколько строк кода, не беда ведь?
   После мы решили, что удалять по одной заметке слишком долго и хотелось бы удалить сразу несколько, а может и все. Ну что же, ещё десяток строк, не беда. Наше приложение становится всё лучше, но нет предела совершенству и мы добавляем ещё немного функционала.
   Одним прекрасным утром, проснувшись и открыв код любимого детища мы обнаруживаем, что прокручивать все 500 с лишним строк стало как-то неудобно и возможно стоило бы разбить код на несколько файлов.

Приняв во внимание данные недостатки, попробуем изначально создать чуть более расширяемую и простую в обслуживании архитектуру для нашего приложения. Создайте отдельную папку для нашего обновлённого приложения и давайте приступать!

##### Для начала создадим файл который будет выполнять функции хранилища ошибок **errors.js**:

1. В файл **errors.js** добавим следующие ошибки:

```js
exports.NOTE_VALIDATION_ERROR = 'Заметка должна содержать заголовок и контент!';
exports.NOTE_NOT_FOUND_ERROR = 'Заметка не найдена!';
exports.NOTE_ALREADY_EXITS_ERROR = 'Заметка с таким заголовком уже существует!';
```

##### Создадим файл **notes.repo.js**. Он будет содержать в себе функции отвечающие за извлечение данных из хранилища:

1. В самом начале файла объявим наши импорты:

```js
const { readFile, writeFile } = require('fs/promises');
const { writeFileSync, existsSync } = require('fs');
const path = require('path');
const { NOTE_NOT_FOUND_ERROR, NOTE_ALREADY_EXITS_ERROR } = require('./errors');
```

Обратите внимание на первую строку. Ранее в своём приложении мы использовали модуль **fs** для работы с файлами с помощью функций обратного вызова. На этот раз мы импортируем **readfile** из модуля **fs/promises** который поставляет набор функций готовых для работы с промисами и асинхронными функциями. Во второй строке мы импортируем синхронные функции проверки существования файла и записи, которые пригодятся ниже. Так же в 3-й строке мы импортируем ошибки, которые понадобятся нам в случае если заметка уже создана или же не найдена.

2. Далее объявим переменную хранящую путь к нашему файлу хранилища и проверку того, что файл хранилища существует:

```js
const STORAGE_FILE_PATH = path.join(__dirname, 'storage.json');

if (!existsSync(STORAGE_FILE_PATH)) {
  const defaultContent = '[]';
  writeFileSync(STORAGE_FILE_PATH, defaultContent);
}
```

Обратите внимание, что проверка выполняется с помощью синхронных функций и происходит в момент запуска программы. Такое поведение гарантирует, что в случае отсутствия, наше хранилище будет создано и заполнено пустым массивом до того, как наши команды начнут своё исполнение.

3. Для извлечения массива всех заметок из хранилища создадим функцию **getAll()**:

```js
exports.getAll = async () => JSON.parse(await readFile(STORAGE_FILE_PATH));
```

Данная функция просто извлекает содержимое нашего json файла и преобразует в обычный javascript массив. Обратите внимание на сколько короче стала запись, по сравнению с версией использующей функции обратного вызова.

> Внимательный человек заметит, что в нашей функции нет обработки ошибок которая присутствует в прошлых версиях. Да, на данном этапе мы не обрабатываем ошибок, которые могут случиться при чтении файла или преобразовании контента в json, но такова изначальная задумка. Обработка ошибок это та вещь, в которой промисы и асинхронные функции действительно хороши, и убедиться в этом вы сможете чуть позже.

4. Для извлечения одной заметки из хранилища - **getOne()**:

```js
exports.getOne = async (noteTitle) => {
  const notes = await this.getAll();
  const note = notes.find((note) => note.title === noteTitle);
  if (!note) {
    throw new Error(NOTE_NOT_FOUND_ERROR);
  }
  return note;
};
```

Обратите внимание, что данная функция использует написанный нами выше **getAll()** для получения массива заметок. Мы пере используем наш код сокращая его базу и делая наши функции более легковесными и читаемыми.

5. Для создания заметки - **create()**:

```js
exports.create = async (note) => {
  const notes = await this.getAll();
  if(notes.some(el => el.title === note.title)) {
    throw new Error(NOTE_ALREADY_EXITS_ERROR);
  }
  notes.push(note);
  await writeFile(STORAGE_FILE_PATH, JSON.stringify(notes));
};
```

6. Для удаления заметки - **delete()**:

```js
exports.deleteOne = async (noteTitle) => {
  const notes = await this.getAll();
  const filtredNotes = notes.filter((note) => note.title !== noteTitle);
  await writeFile(STORAGE_FILE_PATH, JSON.stringify(filtredNotes));
};
```

Отлично, данный файл реализует все функции доступа к данным которые могут нам понадобиться. Можно переходить к следующему этапу.

##### Создадим файл **notes.commands.js** который будет содержать команды доступные нашему приложению.

> Конечно, команды в будущем могут иметь большой размер, свои зависимости, и так далее в зависимости от сложности. В таком случае вам понадобится вынести каждую команду в отдельный файл, а файлы в отдельную папку. Но это уже выходит за рамки нашей статьи, а потому для примера мы поместим всё в один файл.

```js
const { getAll, getOne, create, deleteOne } = require('./notes.repo');

exports.create = async (title, content) => {
  await create({ title, content });
};

exports.list = async () => {
  const notes = await getAll();
  return notes;
};

exports.view = async (title) => {
  const note = await getOne(title);
  const { content } = note;
  return content;
};

exports.remove = async (title) => {
  await deleteOne(title);
};
```

На данный момент наши команды нацелены только на получение данных. Но в случае если нам понадобится добавить какую-либо бизнес-логику, мы без труда может расширить любую из команд(или добавить новые), при этом не думая о форме и месте хранения данных.

##### И наконец создадим файл, который будет содержать логику взаимодействия пользователя с нашим приложением.

Для того чтобы расширить возможности взаимодействия с пользователем воспользуемся пакетом [commander](https://github.com/tj/commander.js). Это один из самых популярных пакетов для создания консольных утилит который имеет большой набор полезных функций.
Для установки в терминале выполните команду npm i commander . После сообщения об успешной установке напишем следующее:

```js
const { program } = require('commander');
const { create, list, view, remove } = require('./notes.commands');
const { NOTE_VALIDATION_ERROR } = require('./errors');

program
  .command('create <title> <content>')
  .description('Создать заметку')
  .action(async (title, content) => {
    await create(title, content);
    console.log('Заметка создана');
  })
  .showHelpAfterError(NOTE_VALIDATION_ERROR);

program
  .command('list')
  .description('Список заметок')
  .action(async () => console.log(await list()));

program
  .command('view <title>')
  .description('Показать заметку')
  .action(async (title) => console.log(await view(title)));

program
  .command('remove <title>')
  .description('Удалить заметку')
  .action(async (title) => {
    await remove(title);
    console.log('Заметка удалена');
  });

program.parseAsync(process.argv).catch((err) => console.log(err.message));
```

Давайте разберём находящийся в файле код:

1. Импортируем program из модуля commander, наши команды, и ошибку валидации(за которую отвечает наш модуль commander).
2. Создаём 4 команды
3. Вызываем функцию которая асинхронно распарсит аргументы командной строки и вызовет необходимую команду.

> Обратите внимание на последнюю строку, а в частности catch(err => console.log(err.message)) . Этот небольшой кусочек кода будет перехватывать все ошибки в нашем приложении, так как в цепочках промисов (которыми под капотом являются и асинхронные функции) любая ошибка будет всплывать по цепочке вызова вверх.

Введите в командную строку node index -h и вы увидите немного магии модуля commander, а именно подсказки о всех доступных нам командах. Теперь наше приложение выглядит более взросло, не так ли?

Итак, мы провели небольшой рефакторинг и улучшили наше приложение сделав его более простым в обслуживании и расширении.  

## Бонус: добавление веб-контролера

После рефакторинга нашего приложения, мы получили возможность свободно изменять/заменять/добавлять функционал на одном из слоёв не касаясь других. Давайте добавим нашему приложению способность работать в вебе по принципу REST api:
1. Для удобства маршрутизации и работы с запросами и ответами установим [Express](https://expressjs.com/) выполнив команду ```npm i express``` . Это один из самых распространённых фреймворков в мире node.js. Если вы знакомы с другими фреймворками, то можете использовать их, или же попробовать обойтись без фреймворка.
2. Создадим файл **bonus-web.js** в который поместим данный код:

```js
const  express  =  require('express');

const { NOTE_VALIDATION_ERROR, NOTE_ALREADY_EXITS_ERROR, NOTE_NOT_FOUND_ERROR } =  require('./errors');

  

const {list, view, create, remove} =  require('./notes.commands');

const  app  =  express();
const  PORT  =  3000;
const  statuses  = {};
statuses[NOTE_VALIDATION_ERROR] =  400;
statuses[NOTE_ALREADY_EXITS_ERROR] =  409;
statuses[NOTE_NOT_FOUND_ERROR] =  404;

app.use(express.json());

const  asyncWrapper  = (fn) => {
	return (req, res, next) => {
	fn(req, res, next).catch(next);
}};

app.get('/notes', asyncWrapper(async (req, res) => {
	const  notes  =  await  list();
	res.json(notes);
}));

app.get('/notes/:id', asyncWrapper(async (req, res) => {
	const  noteTitle  =  req.params.id;
	const  note  =  await  view(noteTitle);
	res.json(note);
}));

app.post('/notes', asyncWrapper(async(req, res) => {
	const { title, content } =  req.body.note;
	if(!title  ||  !content) {
	throw  new  Error(NOTE_VALIDATION_ERROR);
}

await  create(title, content);
	res.json({title, content});
}));
app.delete('/notes/:id', asyncWrapper(async(req, res) => {
	const  noteTitle  =  req.params.id;
	await  remove(noteTitle);
	res.json({ status: 'ok' });
}));

app.use(function (err, req, res, next) {
	const  status  =  statuses[err.message] ||  500;
	res.status(status).json({error: status  ===  500  ? 'internal server error'  :  err.message });
});

app.listen(PORT, () => {
	console.log(`Notes app listening at 	http://localhost:${PORT}`)
});
```
3. Запустим приложение командой  ```node bonus-web```.  Наш сервер запустится по адресу ```localhost:3000 ``` о чём скажет сообщение в консоли: ```Notes app listening at http://localhost:3000``` .  
4.  Проверим работоспособность приложения выполнив команды в терминале git-bash:
  *  ```curl -X POST http://localhost:3000/notes  -H "Content-Type: application/json" -d '{"note": {"title": "test-title", "content": "test-content"}}' ``` - создаст новую заметку. Попробуйте ввести команду ещё раз, и получите ошибку  ```{"error":"Заметка с таким заголовком уже существует!"}``` это значит, что обработка ошибок выполняется корректно.
  * Далее проверим наличие новой заметки в базе. Для этого выполним запрос ```curl http://localhost:3000/notes```  или же откроем вкладку браузера по адресу ```http://localhost:3000/notes```.  Как видно к нашим уже существующим заметкам добавилась новая.  Ради эксперимента можно проделать то же самое с адресом ```http://localhost:3000/notes/test-title```.
  * И наконец удалим нашу тестовую заметку командой ```curl -X DELETE http://localhost:3000/notes/test-title```.  
  * Проверьте ещё раз корректность удаления заметки выполнив GET запрос.
  
  > Обратите внимание, что открыв второе окно терминала вы можете продолжать использовать программу как консольную утилиту, и всё это благодаря тому, что хорошо спроектированное программное обеспечение не должно зависеть от таких вещей как интерфейс взаимодействия с пользователем. 

Итак, мы получили ещё один способ взаимодействия с пользователями для нашего приложения.  Надеюсь данный раздел помог вам понять почему разделение на слои и инкапсуляция логики каждого из них важна при написании приложения.
