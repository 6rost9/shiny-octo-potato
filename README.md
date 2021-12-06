# Особенности `this` в JavaScript

`this` — это ключевое слово, значение которого меняется динамически в зависимости от контекста в котором оно применяется.

Поведение ключевого слова `this` в JavaScript отличается от остальных языков программирования. Например `this` доступен глобально, вызвав `console.log(this);` в консоли браузера мы получим `Window`

```js
    console.log(this);

    // Window {0: Window, window: Window, self: Window, document: document, name: '', location: Location, …}
```

Значение `this` будет зависеть от того как будет вызвана функция, значение `this` в `newFn` будет `Window` не смотря на то, что функция копируются по ссылке.

```js
    const obj = {
        prop: 'Some',
        fn: function() {
            console.log(this);
        }
    }
    
    obj.fn();
    // {prop: 'Some', fn: ƒ}
    
    const newFn = obj.fn;
    newFn();
    // Window {window: Window, self: Window, document: document, name: '', location: Location, …}
```

### Вложенные объекты

Использование this во вложенных объектах может создать путаницу. Стоит помнить о том, что ключевое слово this относиться к тому объекту, в методе которого оно используется.

```js
    const obj = {
        fn: function() {
          console.log(this);
        },
        inside: {
            fn: function(){
                console.log(this);
              }
          }
      };
       

    obj.fn();
    // {inside: {…}, fn: ƒ}

    obj.inside.fn();
    // {fn: ƒ}
```

### Строгий режим

`this` доступен глобально и в методах находящихся вне объекта будет ссылаться на глобальный объект, в случае с браузером это `Window`, но при использовании строго режима `"use strict"` `this` будет иметь значение `undefined`

```js
    "use strict";

    function some() {
        console.log(this);
    }

    some();
    // undefined
```

### Потеря контекста

При знакомстве с языком JavaScript, не очевидна ситуация с потерей контекста и нахождением в `this` объекта, которого мы не ожидали. Передавая метода в качестве аргумента обработчика события или функции для отложенного вызова, произойдет изменение контекста и `this` будет иметь значение отличное от `obj`, не смотря на то, что передали мы его как метод объекта.

```js
    window.onload = function() {
        const obj = {
            prop: 'Some',
            fn: function() {
                console.log(this);
            }
        }

        document.querySelector('button').addEventListener('click', obj.fn)
        // <button>click me!</button>

        setTimeout(obj.fn, 100);
        // Window {window: Window, self: Window, document: document, name: '', location: Location, …}
    };
```

В данном примере этого можно избежать, обернув вызов метода объекта в анонимную функцию

```js
    document.querySelector('button').addEventListener('click', function() {
        obj.fn();
    })
    // {prop: 'Some', fn: ƒ}

    setTimeout(function() {
        obj.fn();
    }, 100);
    // {prop: 'Some', fn: ƒ}
```

### Ключевое слово new и this

Можно найти применение `this` в функциях-конструкторах, для создания объектов, так как оно позволяет работать со множеством объектов, создаваемых с помощью такой функции.

Когда функцию-конструктор вызывают с использованием ключевого слова `new`, `this` в ней указывает на новый объект, который, с помощью конструктора, снабжают свойствами и методами.

```js
    function Person(firstName, lastName) {
        this.firstName = firstName;
        this.lastName = lastName;

        this.sayName = function() {
            console.log(`${this.firstName} ${this.lastName} `);
        }
    }

    const Ivan = new Person('Ivan', 'Petrov');
    Ivan.sayName();
    // Ivan Petrov

    const Vasily = new Person('Vasily', 'Smironv');
    Vasily.sayName();
    // Vasily Smironv
```

## Управление контекстом
### Стрелочные функций

Особенность работы стрелочных функций в том, что у них нет `this` его значение наследуется, таким образом у `fn` объекта `objArrow` `this` примет значение `window`

```js
    const obj = {
        fn: function() {
            console.log(this);
        }
      };
       
    const objArrow = {
        fn: () => console.log(this)
    };
       
    obj.fn();
    // {fn: ƒ}

    objArrow.fn();
    // Window {window: Window, self: Window, document: document, name: '', location: Location, …}
```

### call(), apply(), bind()

JavaScript позволяет устанавливать значение `this`, для этого доступны методы call(), apply(), bind()

Методы call() и apply() позволяют вызвать метод указав для него значение `this`, для этого вызываем call/apply у метода которому хотим задать контекст, указав первым аргументом новое значение `this`, а далее передаем аргументы при необходимости.

По сути методы call() и apply() одинаковы, отличается способ передачи аргументов. Для call() аргументы перечисляются через запятую, а для apply() передаются виде массива.

```js
    function Person(firstName, lastName) {
        this.firstName = firstName;
        this.lastName = lastName;

        this.sayName = function(description = '') {
            console.log(`${this.firstName} ${this.lastName}${description ? ':' : ''} ${description}`);
        }
    }

    const Ivan = new Person('Ivan', 'Petrov');
    
    Ivan.sayName();
    // Ivan Petrov

    Ivan.sayName.call({
        firstName: 'Sergey',
        lastName: 'Belov'
    }, '21 y.o.');
    // Sergey Belov: 21 y.o.

    Ivan.sayName.apply({
        firstName: 'Sergey',
        lastName: 'Belov'
    }, ['21 y.o.']);
    // Sergey Belov: 21 y.o.

    console.log(Ivan);
    // Person {firstName: 'Ivan', lastName: 'Petrov', sayName: ƒ}
```

Метод bind() также дает возможность указать `this`, но при этом возвращая новый метод с привязанным контекстом

```js
    const newIvanSay =  Ivan.sayName.bind({
        firstName: 'Sergey',
        lastName: 'Belov'
    }, '21 y.o.');
    
    newIvanSay();
    // Sergey Belov: 21 y.o.
```
