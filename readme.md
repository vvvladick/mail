Електронна пошта
Спроєктуйте зовнішній інтерфейс для поштового клієнта, що використовує API для відправлення та отримання листів.

Перші кроки

1Завантажте дистрибутив з кодом з https://cdn.cs50.net/web/2020/spring/projects/3/mail.zip і розпакуйте його.
У вашому терміналі виконайте cd у директорію mail.
Виконайте python manage.py makemigrations mail щоб створити міграції для застосунку mail.
Виконайте python manage.py migrate, щоб застосувати міграції до вашої бази даних.
Пояснення
У дистрибутиві знаходиться Django-проєкт під назвою project3, в якому міститься один застосунок mail.

Після того, як ви створите й застосуєте міграції для цього проєкту, виконайте python manage.py runserver щоб розпочати роботу вебсервера. Відкрийте цей сервер у своєму браузері та скористайтеся посиланням «Зареєструватись», щоб зареєструвати новий обліковий запис. Усі листи, які ви будете надсилати й отримувати в межах цього проєкту, будуть зберігатися у вашій базі даних (вони не будуть надсилатися до справжніх поштових серверів), тож ви можете обрати для цього проєкту будь-яку поштову адресу (наприклад, foo@example.com) aі пароль: облікові дані не мусять бути дійсними даними справжньої електронної адреси.

Щойно ви увійдете до облікового запису, ви побачите, що вас перенаправлено на сторінку вхідних листів поштового клієнта; щоправда, поки ця сторінка майже пуста. Натисніть відповідні кнопки, щоб перейти до тек «Надіслані» та «Архів», і зверніть увагу, що вони теж наразі пусті. Натисніть кнопку «Написати», і вас буде перенаправлено до форми, що дозволить вам створити новий лист. Однак коли ви натискаєте на ці кнопки, вас не перенаправляє до нового маршруту і не робиться нового вебзапиту: натомість, увесь цей застосунок – це лише одна сторінка, на якій застосовано JavaScript для контролю за користувацьким інтерфейсом. Давайте тепер уважніше поглянемо у код з дистрибутиву, щоб побачити, як це працює.

Зазирніть у mail/urls.py і зверніть увагу, що типовий маршрут завантажує функцію index у views.py. Тому давайте відкриємо views.py і поглянемо на функцію index. Зверніть увагу, що поки користувач знаходиться в своєму обліковому записі, ця функція виводить шаблон mail/inbox.html. Давайте подивимося на цей шаблон, збережений у mail/templates/mail/inbox.html. Ви помітите, що в тілі сторінки електронна адреса користувача спершу відображається в елементі h2. Після цього на сторінці розміщено кілька кнопок для переходу між різноманітними сторінками застосунку. Зверніть увагу, що ще нижче на сторінці є дві основні секції, кожну з яких визначено елементом div. Перша (з id = emails-view) містить наповнення поштової скриньки (початково пустої). Друга (з id = compose-view) містить форму, де користувач може створити новий лист. Кнопки вздовж верхньої частини мають вибірково показувати і приховувати ці представлення: кнопка «Написати», наприклад, має приховувати emails-view і показувати compose-view; тоді як кнопка «Вхідні» має приховувати compose-view і показувати emails-view.

How do they do that? Зверніть увагу, внизу inbox.html під'єднано файл JavaScript mail/inbox.js. Відкрийте цей файл, його збережено у mail/static/mail/inbox.js, і перегляньте. Зверніть увагу, що після завантаження DOM сторінки, ми прикріплюємо слухачі подій до кожної кнопки. Наприклад, коли натиснуто кнопку inbox ми викликаємо функцію load_mailbox з аргументом 'inbox'; після натискання кнопки compose ми викликаємо функцію compose_email. Що ж роблять ці функції? Функція compose_email спочатку приховує emails-view (встановивши його властивість style.display як none) і показує compose-view (встановивши його властивість style.display як block). AПісля цього функція забирає все, що було в полях введення (де користувач може ввести електронну адресу отримувача, тему і тіло листа), і встановлює їхні значення як пустий рядок '' щоб очистити ці поля. Це означає, що щоразу, як ви натискаєте кнопку «Написати», ви маєте отримувати пусту форму написання листа: ви можете перевірити це, увівши якісь значення у форму, переключивши на «Вхідні», а потім повернувшись до «Написати».

Функція load_mailbox, тим часом, спочатку показує emails-view а потім приховує compose-view. Функція load_mailbox також приймає аргумент, що буде іменем теки з поштою, яку користувач пробує переглянути. Для цього проєкту ви створите поштовий клієнт з трьома теками – «Вхідні» (inbox), «Надіслані» (sent) з усіма надісланими листами і «Архів» (archive) з листами, що були колись в теці «Вхідні», але пізніше їх було заархівовано. Таким чином, аргумент для load_mailbox, буде одним із цих трьох значень, і функція load_mailbox відобразить назву вибраної теки завдяки оновленню innerHTML у emails-view (із заміною першої літери на велику.) Ось чому після вибору певної теки з листами у браузері ви бачите, що назва цієї теки (з великої літери) з’являється у DOM: функція load_mailbox оновлює emails-view, щоб включити необхідний текст.

Звичайно, цей застосунок неповний. Всі теки просто показують назви (Вхідні, Надіслані, Архів), але поки не відображають жодних листів. Немає представлення, щоб побачити наповнення якогось листа. Форма написання дозволяє вам набрати текст листа, але кнопка «Надіслати» поки що нічого не робить. І тепер ваш вихід!

API
Ви будете отримувати й відправляти пошту, а також змінювати статус листів за допомогою API цього застосунку. Ми повністю написали API для вас (і описали його нижче), тож ви можете використовувати його у вашому коді JavaScript. Ми, власне, написали для вас весь код на Python для цього проєкту. Ви повинні завершити цей проєкт за допомогою лише HTML і JavaScript).

Цей застосунок підтримує такі маршрути API:

GET /emails/<str:mailbox>
Надсилання запиту GET до /emails/<mailbox>, де <mailbox> - це inbox, sent, або archive повертатиме вам (у формі JSON) список усіх листів з цієї теки у зворотному хронологічному порядку. Наприклад, якщо ви надішлете запит GET до /emails/inbox, ви можете отримати відповідь JSON подібну до такої (у ній представлено два листи):

[
    {
        "id": 100,
        "sender": "foo@example.com",
        "recipients": ["bar@example.com"],
        "subject": "Привіт!",
        "body": "Привіт, світ!",
        "timestamp": "Січ. 2 2020, 12:00 AM",
        "read": false,
        "archived": false
    },
    {
        "id": 95,
        "sender": "baz@example.com",
        "recipients": ["bar@example.com"],
        "subject": "Зустріч завтра",
        "body": "О котрій зустрічаємось?",
        "timestamp": "Січ. 1 2020, 12:00 AM",
        "read": true,
        "archived": false
    }
]
Зверніть увагу, що для кожного листа зазначено id (унікальний ідентифікатор), sender - електронну адресу відправника, масив отримувачів recipients, рядки для теми subject, тіла листа body, а також позначку часу timestamp, і два булевих значення, що вказують, чи було лист прочитано (read) чи заархівовано (archived).

Як ви могли б отримати доступ до цих значень в JavaScript? Пригадайте, що в JavaScript ви можете використати fetch tщоб зробити вебзапит. А отже, такий код на JavaScript

fetch('/emails/inbox')
.then(response => response.json())
.then(emails => {
    // Вивести листи в консоль
    console.log(emails);

    // ... зробити з листами щось інше ...
});
зробить запит GET до /emails/inbox, конвертує отриману відповідь у JSON і потім представить вам масив листів всередині змінної emails. Ви можете вивести цю змінну в консоль браузера за допомогою console.log (якщо у вашій теці «Вхідні» немає жодного листа, це буде пустий масив) або зробити щось інше з цим масивом.

Також зверніть увагу, що якщо ви зробите запит до недійсної теки (будь-якої, окрім inbox, sent, чи archive), ви отримаєте JSON-відповідь {"error": "Invalid mailbox."}.

GET /emails/<int:email_id>
Відправлення запиту GET до /emails/email_id, де email_id – це ціле число-ідентифікатор листа, поверне JSON-представлення листа на кшталт:

{
        "id": 100,
        "sender": "foo@example.com",
        "recipients": ["bar@example.com"],
        "subject": "Привіт!",
        "body": "Привіт, світ!",
        "timestamp": "Jan 2 2020, 12:00 AM",
        "read": false,
        "archived": false
}
Зверніть увагу, що якщо листа не існує, або якщо у користувача немає доступу до цього листа, маршрут повертатиме помилку 404 «Не знайдено» з JSON-відповіддю {"error": "Email not found."}.

Для прикладу, щоб отримати лист номер 100, ви можете написати такий код на JavaScript:

fetch('/emails/100')
.then(response => response.json())
.then(email => {
    // Вивести лист до консолі
    console.log(email);

    // ... зробити з листом щось інше ...
});
POST /emails
Ми вже розібрались, як отримувати листи – усі листи з теки чи лише один конкретний. Щоб відправити лист, ви можете надіслати запит POST до маршруту /emails. Маршрут вимагає подання трьох фрагментів даних: значення recipients (розділений комами рядок усіх користувачів, яким потрібно надіслати лист), рядка subject і рядка body. Наприклад, ви можете написати JavaScript-код на кшталт:

fetch('/emails', {
  method: 'POST',
  body: JSON.stringify({
      recipients: 'baz@example.com',
      subject: 'Час зустрічі',
      body: 'Може, зустрінемось завтра о 15.00?'
  })
})
.then(response => response.json())
.then(result => {
    // Вивести результат в консоль
    console.log(result);
});
Якщо лист успішно відправлено, маршрут надасть у відповідь код статусу 201 і JSON-відповідь {"message": "Email sent successfully."}.

Зверніть увагу, що обов’язково має бути вказаний принаймні один одержувач: якщо його не надано, маршрут відповість кодом статусу 400 і JSON-відповіддю {"error": "At least one recipient required."}. AТакож усі одержувачі мають бути дійсними користувачами, які зареєструвалися в цьому конкретному вебзастосунку. Якщо ви спробуєте надіслати листа до baz@example.com, але користувача з такою електронною адресою не існує, ви отримаєте JSON-відповідь {"error": "User with email baz@example.com does not exist."}.

PUT /emails/<int:email_id>
Останній маршрут, який вам знадобиться, – це можливість відмічати лист як прочитаний/непрочитаний чи заархівований/розархівований. Щоб зробити це, надішліть запит PUT замість GET до /emails/<email_id>, де email_id – це ідентифікатор листа, який ви пробуєте змінити. Наприклад, JavaScript-код на кшталт:

fetch('/emails/100', {
  method: 'PUT',
  body: JSON.stringify({
      archived: true
  })
})
позначить лист номер 100 як заархівований. Тіло запиту PUT може також бути {archived: false} щоб розархівувати повідомлення, і, за аналогією, може бути або {read: true} або read: false} щоб відповідно позначити лист як прочитаний чи непрочитаний.

Ці чотири маршрути API – отримати всі листи з теки, отримати один лист, відправити лист і змінити статус наявного листа – це всі інструменти, що потрібні вам для завершення проєкту!

Специфікація завдання

За допомогою JavaScript, HTML і CSS завершіть реалізацію вашого односторінкового застосунку – поштового клієнта у inbox.js. Ви маєте врахувати такі вимоги:

Надсилання листа: 

Коли користувач надсилає форму листа, додайте код на JavaScript, щоб справді надіслати лист.
Імовірно, вам захочеться зробити запит POST до /emails, передаючи значення для recipients, subject, і body.
Коли листа буде надіслано, завантажте теку користувача «Надіслані».

Тека:

 Коли користувач заходить до своїх тек «Вхідні», «Надіслані» чи «Архів», завантажте відповідну теку.
o Імовірно, вам захочеться зробити запит GET до /emails/<mailbox>, щоб запитувати листи з конкретної теки.
Коли користувач заходить до теки, застосунок має спочатку зробити запит до API, щоб отримати останні листи з цієї теки.
Коли користувач заходить до теки, назва цієї теки має з’являтися вгорі сторінки (цю частину вже зроблено для вас).
o Після цього кожен лист має бути поданий у власному блоці (наприклад, як <div> з границями), котрий показуватиме відправника, тему і часову позначку листа.
Якщо лист не прочитано, він має бути на білому фоні. Якщо лист прочитано, він має бути на сірому фоні.

Перегляд листа:

 Після натискання на лист користувача має бути скеровано до представлення, де він побачить наповнення цього листа.
Імовірно, вам захочеться зробити запит GET до /emails/<email_id>, щоб отримати лист.
Ваш застосунок має показувати відправника, одержувача, тему, часову позначку й тіло листа.
Вам, певне, захочеться додати ще один div до inbox.html (на додачу до emails-view і compose-view) щоб відобразити лист. Переконайтеся, що ви оновили свій код, щоб приховувати й показувати правильні представлення після натискання на кнопки переходу.
В розділі Підказок зверніть увагу на пункт, що пояснює, як додати слухач подій до елементу HTML, який ви додали до DOM.
Щойно на лист було натиснуто, ви маєте позначити його як прочитаний. Пригадайте, що ви можете надіслати запит PUT до /emails/<email_id>, щоб оновити інформацію про те, чи було лист прочитано, чи ні.

Архівування та розархівування:

 Дозвольте користувачам архівувати та розархівовувати отримані ними листи.
Під час перегляду теки «Вхідні» користувач повинен бачити кнопку, що дозволить йому заархівувати лист. Коли користувач переглядає теку «Архів», у нього повинна бути кнопка, що дозволить розархівувати лист. Ця вимога не стосується листів з теки «Надіслані».
Пригадайте, що ви можете надіслати запит PUT до /emails/<email_id>, щоб позначити лист як заархівований чи розархівований.
Щойно лист було заархівовано чи розархівовано, завантажте теку «Вхідні» користувача.

Відповідь: 

Дозвольте користувачам відповідати на листи.
Під час перегляду листа у користувача повинна бути кнопка «Відповісти», що дозволить йому відповісти на лист.
Коли користувач натискає кнопку «Відповісти», його має бути перенесено до форми написання листа.
Попередньо заповніть адресою відправника recipient початкового листа поле одержувача у формі написання.
Попередньо заповніть рядок теми subject. Якщо початковий лист мав тему foo, нова тема має бути Re: foo. (IЯкщо рядок теми вже починається з Re: , не потрібно додавати його ще раз.)
Попередньо заповніть тіло body листа рядком на кшталт "On Jan 1 2020, 12:00 AM foo@example.com написав:", за яким розмістіть текст початкового листа.

Підказки

Щоб створити елемент HTML і додати до нього слухача подій, ви можете використовувати код JavaScript на кшталт:
const element = document.createElement('div');
element.innerHTML = 'Ось це вміст div.';
element.addEventListener('click', function() {
    console.log('Цей елемент натиснуто!')
});
document.querySelector('#container').append(element);
Цей код створює новий елемент div, встановлює його innerHTML, і додає слухача подій, що викликає конкретну функцію, коли на цей div натискають. Потім він додає новостворений div до елементу HTML з id container (цей код припускає, що елемент HTML з id containerіснує: ви, ймовірно, захочете змінити аргумент querySelector так, щоб він був тим елементом, до якого ви хочете додати елемент).

Вам може знадобитися змінити mail/static/mail/styles.css, щоб додати необхідні для застосунку CSS.
Пригадайте, що коли у вас є масив JavaScript, ви можете створити цикл для кожного елементу цього масиву за допомогою forEach.
Пригадайте, що для запитів POST та PUT Django зазвичай вимагає токен CSRF, щоб захиститися від атак, що підробляють міжсайтові запити. Для цього проєкту ми навмисне вимкнули захист від CSRF в маршрутах API, тож токен вам не знадобиться. Однак у справжніх проєктах завжди краще вберегтися від таких потенційних вразливостей!