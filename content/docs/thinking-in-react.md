---
id: thinking-in-react
title: Философия React
permalink: docs/thinking-in-react.html
redirect_from:
  - 'blog/2013/11/05/thinking-in-react.html'
  - 'docs/thinking-in-react-zh-CN.html'
prev: composition-vs-inheritance.html
---

Нам кажется, React — это отличный способ писать большие и быстрые JavaScript-приложения. Мы в Facebook и Instagram убедились в его хорошей масштабируемости.

Одна из особенностей React — это предлагаемый им процесс мышления при создании приложений. В этом руководстве мы разберём пример создания таблицы продуктов с поиском на React.

## Начнём с макета {#start-with-a-mock}

Представьте, что у вас уже есть JSON API и макет дизайна сайта. Вот как он выглядит: 

![Mockup](../images/blog/thinking-in-react-mock.png)

Наш JSON API возвращает данные, которые выглядят так:

```json
[
  {category: "Sporting Goods", price: "$49.99", stocked: true, name: "Football"},
  {category: "Sporting Goods", price: "$9.99", stocked: true, name: "Baseball"},
  {category: "Sporting Goods", price: "$29.99", stocked: false, name: "Basketball"},
  {category: "Electronics", price: "$99.99", stocked: true, name: "iPod Touch"},
  {category: "Electronics", price: "$399.99", stocked: false, name: "iPhone 5"},
  {category: "Electronics", price: "$199.99", stocked: true, name: "Nexus 7"}
];
```

## Шаг 1: Разобьём интерфейс на составляющие {#step-1-break-the-ui-into-a-component-hierarchy}

Первое, что нужно сделать — выделить отдельные компоненты (и подкомпоненты) в макете и дать им имена. Если вы работаете с дизайнерами, вполне возможно, что они уже как-то называют компоненты — вам стоит пообщаться! Например, слои Photoshop часто подсказывают имена для React-компонентов.

Но как узнать, что стоит делать отдельным компонентом, а что нет? Используйте тот же подход, как при решении создать простую функцию или целый объект. Можно применить [принцип единственной ответственности](https://ru.wikipedia.org/wiki/Принцип_единственной_ответственности): каждый компонент должен заниматься какой-то одной задачей. Если функциональность компонента увеличивается с течением времени, его следует разбить на более мелкие подкомпоненты.

Многие интерфейсы работают с моделью данных JSON. Поэтому хорошо построенная модель, как правило, уже отражает пользовательский интерфейс (а значит, и структуру компонентов). Интерфейс и модели данных часто имеют похожую *информационную архитектуру*, так что разделить интерфейс на части не составляет труда. Разбейте его на компоненты, каждый из которых отображает часть модели данных.

![Component diagram](../images/blog/thinking-in-react-components.png)

Здесь мы видим, что наше приложение состоит из пяти различных компонентов. Курсивом выделены данные, которые эти компоненты представляют.

  1. **`FilterableProductTable` (оранжевый):** контейнер, содержащий пример целиком
  2. **`SearchBar` (синий):** поле *пользовательского ввода*
  3. **`ProductTable` (зелёный):** отображает и фильтрует *список данных*, основанный на *пользовательском вводе*
  4. **`ProductCategoryRow` (голубой):** наименования *категорий*
  5. **`ProductRow` (красный):** отдельно взятый *товар*

Обратите внимание, что внутри `ProductTable` заголовок таблицы ("Name" и "Price") сам по себе отдельным компонентом не является. Отделять его или нет — вопрос личного предпочтения. В данном примере мы решили не придавать этому особого значения и оставить заголовок частью большего компонента `ProductTable`, так как он является всего лишь малой частью общего *списка данных*. Тем не менее, если в будущем заголовок пополнится новыми функциями (например, возможностью сортировать товар), имеет смысл извлечь его в самостоятельный компонент `ProductTableHeader`.

Теперь, когда мы определили компоненты в нашем макете, давайте расположим их согласно иерархии. Компоненты, которые являются частью других компонентов, в иерархии отображаются как дочерние:

  * `FilterableProductTable`
    * `SearchBar`
    * `ProductTable`
      * `ProductCategoryRow`
      * `ProductRow`

## Шаг 2: Создадим статическое приложение в React {#step-2-build-a-static-version-in-react}

<p data-height="600" data-theme-id="0" data-slug-hash="BwWzwm" data-default-tab="js" data-user="lacker" data-embed-version="2" class="codepen">Пример кода <a href="https://codepen.io/gaearon/pen/BwWzwm">Философия React: Шаг 2</a> на <a href="https://codepen.io">CodePen</a>.</p>

<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

Теперь, когда все компоненты расположены в иерархическом порядке, пришло время воплотить в жизнь наше приложение. Самый лёгкий способ — создать версию, которая использует модель данных и рендерит интерфейс, но не предполагает никакой интерактивности. Разделять эти процессы полезно. Написание статического приложения требует много печатания и совсем немного мышления. С другой стороны, создание интерактивного приложения подразумевает более глубокий мыслительный процесс и лишь долю рутинной печати. Позже мы разберёмся, почему так получается.

Чтобы построить статическое приложение, отображающее модель данных, нам нужно создать компоненты, которые используют другие компоненты и передают данные через *пропсы*. *Пропсы* — это способ передачи данных от родителя к потомку. Если вы знакомы с понятием *состояния*, то для статического приложения это как раз то, чего вам **использовать не нужно**. Состояние подразумевает собой данные, которые меняются со временем — интерактивность. Так как мы работаем над статическим приложением, нам этого не нужно.

Написание кода можно начать как сверху вниз (с большого `FilterableProductTable`), так и снизу вверх (с маленького `ProductRow`). Более простые приложения удобнее начать с компонентов, находящихся выше по иерархии. В более сложных приложениях удобнее в первую очередь создавать и тестировать подкомпоненты.

В конце этого шага у вас на руках появится библиотека повторно используемых компонентов, отображающих вашу модель данных. Так как это статическое приложение, компоненты будут иметь только методы `render()`. Компонент выше по иерархии (`FilterableProductTable`) будет передавать модель данных через пропсы. Если вы внесёте изменения в базовую модель данных и снова вызовете `ReactDOM.render()`, то пользовательский интерфейс отразит эти изменения. Вы можете увидеть, как обновляется интерфейс и где следует сделать очередные изменения. Благодаря **одностороннему потоку данных** (или *односторонней привязке*), код работает быстро, но остаётся понятным. 

Если у вас остались вопросы по выполнению данного шага, обратитесь к [документации React](/docs/).

### Небольшое отступление: чем пропсы отличаются от состояния {#a-brief-interlude-props-vs-state}

Существует два типа "модели" данных в React: пропсы и состояние. Важно, чтобы вы понимали разницу между ними, иначе обратитесь к [официальной документации React](/docs/state-and-lifecycle.html). Посмотрите также раздел [Какая разница между state и props?](/docs/faq-state.html#what-is-the-difference-between-state-and-props)

## Шаг 3: Определим минимальное (но полноценное) отображение состояния интерфейса {#step-3-identify-the-minimal-but-complete-representation-of-ui-state}

Чтобы сделать наш UI интерактивным, нужно, чтобы модель данных могла меняться со временем. В React это возможно с помощью **состояния**.

Чтобы правильно построить приложение, сначала нужно продумать необходимый набор данных изменяемого состояния. Главное тут следовать принципу разработки [DRY: *Don't Repeat Yourself* (рус. не повторяйся)](https://ru.wikipedia.org/wiki/Don%E2%80%99t_repeat_yourself). Определите минимально необходимое состояние, которое нужно вашему приложению, всё остальное вычисляйте при необходимости. Например, если вы создаёте список дел, держите массив пунктов списка под рукой — но не стоит хранить отдельное состояние для количества дел в списке. Если надо отобразить количество элементов – используйте длину существующего массива.

Давайте перечислим все данные в нашем демо-приложении. У нас есть:

  * Первоначальный список товаров.
  * Поисковый запрос, введённый пользователем.
  * Значение чекбокса.
  * Отфильтрованный список товаров.

Давайте рассмотрим данные по частям и определим, что должно храниться в состоянии. Задайте себе три вопроса:

  1. Передаются ли они от родителя через пропсы? Если так, тогда эти данные не должны храниться в состоянии компонента.
  2. Остаются ли они неизменными со временем? Если так, тогда их тоже не следует хранить в состоянии.
  3. Можете ли вы вычислить их на основании любых других данных в своём компоненте или пропсов? Если так, тогда это тоже не состояние.

Исходный список товаров передаётся через пропсы, так что не нужно хранить его в состоянии компонента. Поисковый запрос и состояние чекбокса изменяются со временем, и их нельзя вычислить из других данных, так что они вполне сойдут за состояние. Напоследок, отфильтрованный список товаров не является состоянием, так как его можно вычислить из оригинального списка, поискового запроса и значения чекбокса.

В итоге, состоянием являются:

  * Поисковый запрос, введённый пользователем
  * Значение чекбокса

## Шаг 4: Определим, где должно находиться наше состояние{#step-4-identify-where-your-state-should-live}

<p data-height="600" data-theme-id="0" data-slug-hash="qPrNQZ" data-default-tab="js" data-user="lacker" data-embed-version="2" class="codepen">Пример кода <a href="https://codepen.io/gaearon/pen/qPrNQZ">Философия React: Шаг 4</a> на <a href="https://codepen.io">CodePen</a>.</p>

Итак, мы определили минимальный набор состояний приложения. Далее нам нужно выяснить, какой из компонентов *владеет* состоянием или изменяет его.

Помните: в React поток данных односторонний и сходит сверху вниз в иерархическом порядке. Сначала может быть не совсем ясно, какой из компонентов какое состояние должен хранить. **На этом этапе новички спотыкаются чаще всего.** Чтобы разобраться, следуйте этим инструкциям:

Для каждой части состояния в приложении:

  * Определите компоненты, которые рендерят что-то исходя из этого состояния.
  * Найдите общий главенствующий компонент (компонент, расположенный над другими компонентами, которым нужно это состояние). 
  * Либо общий главенствующий компонент, либо любой компонент, стоящий выше по иерархии, должен содержать состояние.
  * Если вам не удаётся найти подходящий компонент, то создайте новый исключительно для хранения состояния и разместите его выше в иерархии над общим главенствующим компонентом.

Давайте применим эту стратегию на примере нашего приложения:

  * Задача `ProductTable` — отфильтровать список товаров, основываясь на состоянии, а `SearchBar` — отобразить состояние для поискового запроса и чекбокса.
  * Общий главенствующий компонент для обоих — `FilterableProductTable`.
  * По идее, имеет смысл содержать текст фильтра и значение чекбокса в `FilterableProductTable`.

Итак, мы приняли решение расположить наше состояние в `FilterableProductTable`. Первое, что нужно сделать — добавить свойство `this.state = {filterText: '', inStockOnly: false}` в конструктор `FilterableProductTable`, чтобы отобразить начальное состояние нашего приложения. После этого передайте `filterText` и `inStockOnly` в `ProductTable` и `SearchBar` через пропсы. Напоследок, используйте пропсы для фильтрации строк в `ProductTable` и определения значений полей формы `SearchBar`.

Вы заметите изменения в поведении вашего приложения: задайте значение `"ball"` для `filterText` и обновите страницу. Вы увидите соответствующие изменения в таблице данных.

## Шаг 5: Добавим обратный поток данных {#step-5-add-inverse-data-flow}

<p data-height="600" data-theme-id="0" data-slug-hash="LzWZvb" data-default-tab="js,result" data-user="rohan10" data-embed-version="2" data-pen-title="Thinking In React: Step 5" class="codepen">Пример кода <a href="https://codepen.io/gaearon/pen/LzWZvb">Философия React: Шаг 5</a> на <a href="https://codepen.io">CodePen</a>.</p>

Пока что наше приложение рендерится в зависимости от пропсов и состояния, передающихся вниз по иерархии. Теперь мы обеспечим поток данных в обратную сторону: наша задача сделать так, чтобы компоненты формы в самом низу иерархии обновляли состояние в `FilterableProductTable`.

Поток данных в React — однонаправленный. Это помогает понять, как работает приложение, но нам потребуется немного больше кода, чем с традиционной двусторонней привязкой данных.

Если вы попытаетесь ввести текст в поле поиска или установить флажок в чекбоксе данного примера, то увидите, что React игнорирует любой ввод. Это закономерно, так как ранее мы приравняли значение пропа `value` в `input` к `state` в `FilterableProductTable`.

Давайте подумаем, как мы хотим изменить поведение. Нам нужно, чтобы при изменениях поисковой формы менялось состояние ввода. Так как компоненты должны обновлять только относящееся к ним состояние, `FilterableProductTable` передаст колбэк в `SearchBar`. В свою очередь, `SearchBar` будет вызывать этот колбэк каждый раз, когда надо обновить состояние. Чтобы получать уведомления об изменениях элементов формы, мы можем использовать событие `onChange`. Колбэки, переданные из компонента `FilterableProductTable`, вызовут `setState()`, и приложение обновится.

## Вот и всё {#and-thats-it}

Надеемся, что этот пример поможет вам получить представление о том, как подойти к созданию компонентов и приложений в React. Хотя этот процесс и использует немного больше кода, помните, что код читают чаще, чем пишут. А модульный и прямолинейный код читается значительно легче. Когда вы начнёте создавать большие библиотеки компонентов, вы сможете по-настоящему оценить прямолинейность и связанность React, а повторно используемые компоненты сделают ваш код намного меньше. :)
