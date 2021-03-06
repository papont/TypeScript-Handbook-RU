# Введение

С момента введения классов в TypeScript и ES6 существуют определённые случаи, требующие дополнительных функций для поддержки аннотирующих или модифицирующих классов, а также членов класса.
Декораторы предоставляют способ добавления аннотаций и метапрограммного синтаксиса для объявлений класса, их членов.
Декораторы -- это [предложение 1-ой стадии](https://github.com/wycats/javascript-decorators/blob/master/README.md) для JavaScript и экспериментальная функциональность TypeScript.

> ЗАМЕТКА&emsp; Декораторы -- экспериментальная функция, которая может измениться в будущих релизах.

Чтобы включить экспериментальную поддерржку декораторов Вы должны включить опцию компилятора `experimentalDecorators` либо в командной строке, либо в файле `tsconfig.json`.

**Командная Строка**:

```shell
tsc --target ES5 --experimentalDecorators
```

**tsconfig.json**:

```json
{
    "compilerOptions": {
        "target": "ES5",
        "experimentalDecorators": true
    }
}
```

# Декораторы

*Декоратор* -- это особый вид объявления, который может быть прикреплён к [объявлению класса](#Декораторы-Класса), [методу](#Декораторы-Методов), [акцессору](#Декоратор-Акцессора), [свойству](#Декораторы-Свойства) или [параметру](#Декораторы-Параметров).
Декораторы используют форму `@expression`, где `expression` должно вычислиться в функцию, которая будет вызвана во время исполнения, вместе с информацией декорированного объявления.

Например, учитывая декоратор `@sealed` мы могли бы написать функцию `sealed` следующим образом:

```ts
function sealed(target) {
    // что-то сделать с переменной 'target' ...
}
```

> ЗАМЕТКА&emsp; Увидеть более подробный пример декоратора можно в разделе [Декораторы Класса](#Декораторы-Класса).

## Фабрика Декораторов

Если мы захотим настроить то, как декоратор применяется к объявлению, мы можем написать фабрику декораторов.
*Фабрика Декораторов* -- это функция, возвращающая выражение, которая будет вызвана декоратором во время исполнения программы.

Мы можем написать фабрику декораторов следующим образом:

```ts
function color(value: string) { // это фабрика декораторов
    return function (target) { // это декоратор
        // сделать что-то с переменными 'target' и 'value'...
    }
}
```

> ЗАМЕТКА&emsp; Более подробный пример фабрики декораторов можно увидеть в разделе [Декораторы Методов](#Декораторы-Методов).

## Композиция Декоратора

К объявлению могут быть применены несколько декораторов, как в следующем примере:

* На одной строке:

  ```ts
  @f @g x
  ```

* На несколько строк:

  ```ts
  @f
  @g
  x
  ```

Когда несколько декораторов применяются к одному объявлению, их вычисление похоже на [композицию функции в математике](http://en.wikipedia.org/wiki/Function_composition). В этой модели, при композиции функций *f* и *g*, результирующий композит (*f* ∘ *g*)(*x*) эквивалентен *f*(*g*(*x*)).

Так, при вычислении нескольких декораторов на одном объявлении, выполняются следующие шаги:

1. Выражения для каждого декоратора вычисляются сверху вниз.
2. Результаты затем вызываются как функции снизу вверх.

Если мы будем использовать [фабрику декораторов](#Фабрика-Декораторов), мы можем наблюдать этот порядок вычисления вместе с следующим примером:

```ts
function f() {
    console.log("f(): вычисляется");
    return function (target, propertyKey: string, descriptor: PropertyDescriptor) {
        console.log("f(): вызван");
    }
}

function g() {
    console.log("g(): вычисляется");
    return function (target, propertyKey: string, descriptor: PropertyDescriptor) {
        console.log("g(): вызван");
    }
}

class C {
    @f()
    @g()
    method() {}
}
```

Что вывело бы в консоль:

```shell
f(): вычисляется
g(): вычисляется
g(): вызван
f(): вызван
```

## Вычисление Декоратора

Есть определённый порядок применения декораторов к различным объявлениям внутри класса:

1. *Декораторы Параметров* с последующими *Декораторами Методов*, *Акцессоров* или *Свойств* применяются для каждого члена экземпляра.
2. *Декораторы Параметров* с последующими *Декораторами Методов*, *Акцессоров* или *Свойств* применяются для каждого статического члена.
3. *Декораторы Параметров* применяются для конструкторов.
4. *Декораторы Класса* применяются для класса.

## Декораторы Класса

*Декоратор Класса* объявляется перед объявлением класса.
Декоратор класса применяется к конструктору класса и может быть использован для наблюдения, модифицирования или замещения определения класса.
Декоратор класса не может быть использован в файле объявлений или в любом другом окружающем контексте (например, в классе `declare`).

Выражение декоратора класса будет вызываться как функция во время исполнения, а конструктор отдекарированного класса -- как его единственный аргумент.

Если декоратор класса возвратит значение, он заменит объявление класса с помощью предоставленного конструктора.

> ЗАМЕТКА&nbsp; Если Вы решите вернуться к новому конструктору, тогда Вы должны позаботиться о сохранении исходного прототипа.
Логика, использующая декораторы во время выполнения программы, не сделает это за Вас.

Дальше -- пример декоратора класса (`@sealed`), применённый к классу `Greeter`:

```ts
@sealed
class Greeter {
    greeting: string;
    constructor(message: string) {
        this.greeting = message;
    }
    greet() {
        return "Привет, " + this.greeting;
    }
}
```

Мы можем определить декоратор `@sealed`, используя следующее объявление функции:

```ts
function sealed(constructor: Function) {
    Object.seal(constructor);
    Object.seal(constructor.prototype);
}
```

Когда декоратор `@sealed` будет выполнен, он 'запечатает' (seal - печать) конструктор, и его прототип.

## Декораторы Методов

*Декоратор Метода* объявляется перед объявлением метода.
Декоратор применяется к *Дескриптору Свойства* метода, также может быть использован для наблюдения, модифицирования или замещения определения метода.
Декоратор метода не может быть использован в файле объявления, при перегрузке или в любом другом окружающем контексте (например, в классе `declare`).

Выражение для декоратора метода будет вызываться как функция во время исполнения программы со следующими тремя аргументами:

1. Либо конструктор класса для статичного члена, либо прототип класса для члена экземпляра.
2. Имя члена.
3. *Дескриптор Свойства* члена.

> ЗАМЕТКА&emsp; *Дескриптор Свойства* будет `undefined`, если цель Вашего скрипта меньше, чем стандарт `ES5`.

Если декоратор метода возвратит значение, это значение будет использоваться как *Дескриптор Свойства* для метода.

> ЗАМЕТКА&emsp; Возвращаемое значение игнорируется, если цель вашего скрипта меньше, чем стандарт `ES5`.

Дальше -- пример декоратора метода (`@enumerable`), применённый к методу класса `Greeter`:

```ts
class Greeter {
    greeting: string;
    constructor(message: string) {
        this.greeting = message;
    }

    @enumerable(false)
    greet() {
        return "Привет, " + this.greeting;
    }
}
```

Мы можем определить декоратор `@enumerable`, используя следующее объявление функции:

```ts
function enumerable(value: boolean) {
    return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
        descriptor.enumerable = value;
    };
}
```

Декоратор `@enumerable(false)` -- это [фабричный декоратор](#Фабрика-Декораторов).
Когда будет вызван декоратор `@enumerable(false)`, он изменит свойство `enumerable` дескриптора свойства.

## Декоратор Акцессора

*Декоратор Акцессора* объявляется перед объявлением акцессора.
Декоратор акцессора применяется к *Дескриптору Свойства* и может быть использован для наблюдения, модифицирования или замещения определения акцессора.
Декоратор акцессора не может быть использован в файле объявления или в любом другом окружающем контексте (например, класс `declare`).

> ЗАМЕТКА&emsp; TypeScript запрещает декорирование `get` и `set` акцессоров для одного члена.
Вместо этого, все декораторы члена должны быть применены к первому акцессору, указанному в текстовом порядке.
Это потому, что декораторы применяются к *Дескриптору Свойства*, который комбинирует оба метода доступа `get` и `set`, а не каждое объявление в отдельности.

Выражение для декоратора акцессора будет вызвано как функция во время исполнения со следующими тремя аргументами:

1. Либо конструктор класса для статичного члена, либо прототип класса для члена экземпляра.
2. Имя члена.
3. *Дескриптор Свойства* для члена.

> ЗАМЕТКА&emsp; *Дескриптор Свойства* будет `undefined`, если цель вашего скрипта меньше, чем стандарт `ES5`.

Если декоратор акцессора возвратит значение, это значение будет использоваться как *Дексриптор Свойства* для члена.

> ЗАМЕТКА&emsp; Возвращаемое значение игнорируется, если цель вашего скрипта меньше, чем стандарт `ES5`.

Дальше -- пример декоратора акцессора (`@configurable`), применённый к члену класса `Point`:

```ts
class Point {
    private _x: number;
    private _y: number;
    constructor(x: number, y: number) {
        this._x = x;
        this._y = y;
    }

    @configurable(false)
    get x() { return this._x; }

    @configurable(false)
    get y() { return this._y; }
}
```

Мы можем определить декоратор `@configurable`, используя следующее объявление функции:

```ts
function configurable(value: boolean) {
    return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
        descriptor.configurable = value;
    };
}
```

## Декораторы Свойства

*Декоратор Свойства* объявляется перед объявлением свойства.
Декоратор свойства не может быть использован в файле объявления или в любом другом окружающем контексте (например, в классе `declare`).

Выражение декоратора свойства будет вызвано как функция во время исполнения со следующими аргументами:

1. Либо конструктор класса для статичного члена, либо прототип класса для члена экземпляра.
2. Имя члена.

> ЗАМЕТКА&emsp; *Дескриптор Свойства* не предоставляется в качестве аргумента для декоратора свойства из-за того, как декораторы свойства инициализируются в TypeScript.
Это потому, что на текущий момент не существует механизма описать свойство экзмепляра при определении членов прототипа, также нет способа наблюдения или модифицирования инициализатора для свойства.
Так, дескриптор свойства может быть использован только для наблюдения того, что для класса было объявлено свойство.

Если декоратор свойства возвращает значение, это значение будет использоваться в качестве *Дескриптора Свойства* для члена.

> ЗАМЕТКА&emsp; Возвращаемое значение игнорируется, если цель вашего скрипта меньше, чем стандарт `ES5`.

Мы можем использовать эту информцию для записи метаинформации о свойстве, как в следующем примере:

```ts
class Greeter {
    @format("Hello, %s")
    greeting: string;

    constructor(message: string) {
        this.greeting = message;
    }
    greet() {
        let formatString = getFormat(this, "greeting");
        return formatString.replace("%s", this.greeting);
    }
}
```

Мы можем затем определить декоратор `@format` и `getFormat`, используя следующие объявления функции:

```ts
import "reflect-metadata";

const formatMetadataKey = Symbol("format");

function format(formatString: string) {
    return Reflect.metadata(formatMetadataKey, formatString);
}

function getFormat(target: any, propertyKey: string) {
    return Reflect.getMetadata(formatMetadataKey, target, propertyKey);
}
```

Декоратор `@format("Привет, %s")` [фабричный декоратор](#Фабрика-Декораторов).
При вызове `@format("Привет, %s")`, декоратор добавляет метаданные для свойства используя функцию `Reflect.metadata` из библиотеки `reflect-metadata`.
При вызове `getFormat`, функция считывает значение метаданных.

> ЗАМЕТКА&emsp; Этот пример требует библиотеки `reflect-metadata`.
Смотрите раздел [Метаданные](#Метаинформация) для большей информации о библиотеке `reflect-metadata`.

## Декораторы Параметров

*Декоратор Параметра* объявляется перед объявлением параметра.
Декоратор параметра применяется к объявлению конструктора или методу класса.
Декоратор параметра не может быть использован в файле объявлений, при перегрузке или в любом другом окружающем контексте (например, в классе `declare`).

Выражение декоратора параметра будет вызвано как функция во время исполнения, со следующими тремя аргументами:

1. Либо конструктор класса для статичного члена, либо прототип класса для члена экземпляра.
2. Имя члена.
3. Порядковый индекс параметра в списке параметров функции.

> ЗАМЕТКА%emsp; Декоратор параметра может быть использован только для наблюдения того, что параметр был объявлен у метода.

Возвращаемое значение декоратора параметра игнорируется.

Дальше -- пример декоратора параметра (`@required`), применённый к параметру члена класса `Greeter`:

```ts
class Greeter {
    greeting: string;

    constructor(message: string) {
        this.greeting = message;
    }

    @validate
    greet(@required name: string) {
        return "Привет " + name + ", " + this.greeting;
    }
}
```

Мы затем можем определить декораторы `@required` и `@validate`, используя следующие объявления функций:

```ts
import "reflect-metadata";

const requiredMetadataKey = Symbol("required");

function required(target: Object, propertyKey: string | symbol, parameterIndex: number) {
    let existingRequiredParameters: number[] = Reflect.getOwnMetadata(requiredMetadataKey, target, propertyKey) || [];
    existingRequiredParameters.push(parameterIndex);
    Reflect.defineMetadata(requiredMetadataKey, existingRequiredParameters, target, propertyKey);
}

function validate(target: any, propertyName: string, descriptor: TypedPropertyDescriptor<Function>) {
    let method = descriptor.value;
    descriptor.value = function () {
        let requiredParameters: number[] = Reflect.getOwnMetadata(requiredMetadataKey, target, propertyName);
        if (requiredParameters) {
            for (let parameterIndex of requiredParameters) {
                if (parameterIndex >= arguments.length || arguments[parameterIndex] === undefined) {
                    throw new Error("Отсутствуют требуемые аргументы.");
                }
            }
        }

        return method.apply(this, arguments);
    }
}
```

Декоратор `@required` добавляет метаинформацию, которая помечает параметр как обязательный.
Декоратор `@validate` затем обёртывает существующий метод `greet` в функцию, которая проводит проверку аргументов перед вызовом исходного метода.

> ЗАМЕТКА&emsp; Этот пример требует библиотеку `reflect-metadata`.
Смотрите [метаинформация](#metadata) для большей информации о библиотеке `reflect-metadata`.

## Метаинформация

Некоторые примеры используют библиотеку `reflect-metadata`, которая добавляет полифилл для [экспериментального API метаданных](https://github.com/rbuckton/ReflectDecorators).
Эта библиотека ещё не является частью стандарта ECMAScript (JavaScript).
Однако после того, как декораторы официально примут в состав стандарта ECMAScript, эти расширения будут предложены к принятию.

Вы можете установить эту библиотеку через npm:

```shell
npm i reflect-metadata --save
```

TypeScript включает экспериментальную поддержку генерации определённых типов метаданных для объявлений, у которых есть декоратор.
Чтобы включить эту экспериментальную поддержку, Вы должны установить опцию компилятора `emitDecoratorMetadata` либо в командной строке, либо в вашем `tsconfig.json`.

**Командная Строка**:

```shell
tsc --target ES5 --experimentalDecorators --emitDecoratorMetadata
```

**tsconfig.json**:

```json
{
    "compilerOptions": {
        "target": "ES5",
        "experimentalDecorators": true,
        "emitDecoratorMetadata": true
    }
}
```

Когда эта опция включена, во время исполнения будет доступна информация о типе, при условии, что импортирована библиотека `reflect-metadata`.

Мы можем увидеть это в действии в следующем примере:

```ts
import "reflect-metadata";

class Point {
    x: number;
    y: number;
}

class Line {
    private _p0: Point;
    private _p1: Point;

    @validate
    set p0(value: Point) { this._p0 = value; }
    get p0() { return this._p0; }

    @validate
    set p1(value: Point) { this._p1 = value; }
    get p1() { return this._p1; }
}

function validate<T>(target: any, propertyKey: string, descriptor: TypedPropertyDescriptor<T>) {
    let set = descriptor.set;
    descriptor.set = function (value: T) {
        let type = Reflect.getMetadata("design:type", target, propertyKey);
        if (!(value instanceof type)) {
            throw new TypeError("Недопустимый тип.");
        }
    }
}
```

TypeScript компилятор внедрит информацию о типе, используя декоратор `@Reflect.metadata`.
Вы могли бы его считать эквивалентным следующему коду:

```ts
class Line {
    private _p0: Point;
    private _p1: Point;

    @validate
    @Reflect.metadata("design:type", Point)
    set p0(value: Point) { this._p0 = value; }
    get p0() { return this._p0; }

    @validate
    @Reflect.metadata("design:type", Point)
    set p1(value: Point) { this._p1 = value; }
    get p1() { return this._p1; }
}

```

> ЗАМЕТКА&emsp; Декоратор метаинформации -- это экспериментальная функциональность, а также может внести критические изменения в будущих релизах.

[Источник](http://typescript-lang.ru/docs/Decorators.html)