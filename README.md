# Стандарт AST - abstract syntax tree

## Общее

Кодировка файла `cp1251`

## Команды
|Num| Name             |Type    | Description |
|:-:|:-----------------|:------:|:------------|
| 1 | CMD_SEPARATOR    | LIST   | Разделитель команд. Имитирует список. Левый потомок - команда, правого или нет, или такой же разделитель
| 2 | VAR_DEFINITION   | BINARY | Определение переменной. Слева лист типа переменная, справа либо ничего, либо выражение
| 3 | CONST_VAR_DEF    | UNARY  | Опциональный родитель VAR_DEFINITION и ARRAY_DEFINITION
| 4 | ARRAY_DEFINITION | BINARY | Определение массива. Слева поддерево: (VAR_SEPARATOR (переменная) (константное выражение - индекс)). Справа либо ничего, либо список выражений через VAR_SEPARATOR
| 5 | FUNC_DEFINITION  | BINARY | Определение функции. Слева поддерево: (VAR_SEPARATOR (переменная) (поддерево аргументов (список из VAR_SEPARATOR))). Справа список команд через CMD_SEPARATOR
| 6 | ASSIGNMENT       | BINARY | Присваивание. Слева лист переменная, справа выражение
| 7 | ASSIGNMENT_ADD   | BINARY |
| 8 | ASSIGNMENT_SUB   | BINARY |
| 9 | ASSIGNMENT_MUL   | BINARY |
|10 | ASSIGNMENT_DIV   | BINARY |
|11 | ARRAY_ELEM       | BINARY | Элемент массива. Слева лист переменная - имя массива, справа выражение - индекс элемента
|15 | VAR_SEPARATOR    | LIST   | Имитатор списка для аргументов функции и т.п.
|16 | FUNC_CALL        | BINARY | Вызов функции. Слева лист переменная, справа список выражений через VAR_SEPARATOR
|17 | RETURN           | UNARY  | Возврат из функции. Слева ничего, справа выражение
|20 | MATH_ADD         | BINARY | Сложение
|21 | MATH_SUB         | BINARY | Вычитание
|22 | MATH_MUL         | BINARY | Умножение
|23 | MATH_DIV         | BINARY | Деление
|24 | MATH_SQRT        | UNARY  | Корень
|25 | MATH_SIN         | UNARY  | Синус
|26 | MATH_COS         | UNARY  | Косинус
|27 | MATH_NEGATIVE    | UNARY  | Унарный минус
|28 | MATH_DIFF        | BINARY | Оператор дифференцирования Слева выражение, справа лист-переменная с номером переменной, по которой дифференцируем
|40 | LOGIC_GREAT      | BINARY | >
|41 | LOGIC_LOWER      | BINARY | <
|42 | LOGIC_NOT_EQUAL  | BINARY | !=
|43 | LOGIC_EQUAL      | BINARY | ==
|44 | LOGIC_GREAT_EQ   | BINARY | >=
|45 | LOGIC_LOWER_EQ   | BINARY | <=
|50 | PREFIX_ADD       | BINARY | ++x <br> 1) Слева обязательно переменная, справа, либо следующий препост-оператор, либо ничего, либо переменная (последние два варианта означают одно и то же). В таком списке операторов все переменные должны иметь один номер. Такое дублирование сделано для оптимального чтения на бекенде; <br> 2) Сначала только префиксные операторы, потом только постфиксные. То есть префиксный не может быть потомком постфиксного. <br><br> Аналогично для всех препост-операторов
|51 | PREFIX_SUB       | BINARY | --x
|52 | POSTFIX_ADD      | BINARY | x++
|53 | POSTFIX_SUB      | BINARY | x--
|60 | WHILE            | BINARY | while. Слева вычисляемое выражение, справа либо список команд, либо ELSE
|61 | DO_WHILE         | BINARY | do {} while (). Аналогично, ELSE нельзя
|63 | IF               | BINARY | Аналогично
|64 | DO_IF            | BINARY | do {} if () - не спрашивайте, можете забить
|66 | ELSE             | BINARY | Слева список команд, если выполняется основная ветвь, справа если else ветвь
|67 | BREAK            | LEAF   | break
|68 | CONTINUE         | LEAF   | continue
|69 | NEW_SCOPE        | UNARY  | Новая область видимости переменных. Слева ничего, справа список команд
|70 | IN               | LEAF   | команда ассемблера `in`
|71 | OUT              | UNARY  | команда ассемблера `out`. Слева ничего, справа выражение
|72 | SHOW             | LEAF   | команда ассемблера `shw`
|73 | SET_FPS          | UNARY  | команда ассемблера `fps`. Слева ничего, справа выражение, которое можно вычислить во время компиляции (константное)

## Типы узлов

- `1` - оператор. Значение - номер оператора
- `2` - число. Значение - число
- `3` - переменная. Значение - номер переменной

## Формат

Инфиксная запись

Каждый узел характеризуется дебаг информацией (`DebugInfo`), типом, значением и потомками. Если нет потомка, что `(nil)`

```(DebugInfo, <тип>, <значение>, <левый потомок> <правый потомок>)```

E.g.
```({"main.code", 1, 2, 3}, 0, 0, (nil) (nil))```


### Формат `DebugInfo`

```
{
    <путь до исходника>,
    <номер строки>,
    <номер символа с начала строки>,
    <номер первого символа строки с начала файла>
}
```

## Файл

Сначала идёт дерево. Затем `\n` и список имён переменных (каждое имя на новой строке). Нумерация с 0, соответствует порядку

E.g.
```
({"main.code", 1, 2, 3}, 0, 0, (nil) (nil))
x
y
z

```

# Стандарт IR - intermediate representation

Формат представляет собой односвязный список блоков. Каждый блок имеет тип, параметры и индекс следующего блока.

## Формат блока `IRNode`

```
struct IRNode {
    IRNodeType type = IRNodeType::NONE;

    IRVal src[2]  = {{}, {}};
    IRVal dest = {};

    union {
        MathOper math = MathOper::NONE;
        CmpType cmp;
        JmpType jmp;
    } subtype;

    DebugInfo debug_info = {};
};
```

## Типы блоков `IRNodeType`

| Num | Name                      |   src[0]           |   src[1]        |   dest       |  subType   | Description |
|:---:|:--------------------------|:------------------:|:---------------:|:------------:|:----------:|:------------|
|  0  | NONE                      |                    |                 |              |            | Empty block. May be used for jump destination
|  1  | START                     |                    |                 |              |            | Entry point beginning
|  2  | END                       |                    |                 |              |            | Entry point end
|  3  | BEGIN_FUNC_DEF            | local vars number  |                 |              |            | Function definition beginning
|  4  | END_FUNC_DEF              |                    |                 |              |            | Function definition end
|  5  | CALL_FUNC                 | local vars number  |                 | func block i |            | Function call
|  6  | RET                       |                    |                 |              |            | Return from function
|  7  | INIT_MEM_FOR_GLOBALS      | global vars number |                 |              |            | Global scope variables memory and lib functions init
|  8  | COUNT_ARR_ELEM_ADDR_CONST | offset             |                 |              |            | Count address of array element
|  9  | ARR_ELEM_ADDR_ADD_INDEX   | index source       | global or local |              |            | Add value from stack to address of array element
| 10  | MOV                       | source             | special data    | destination  |            | Mov value from src[0] to dest (stack, memory, register)
| 11  | SWAP                      | operand 1          | operand 2       |              |            | Swap 2 values from src[0] and src[1]
| 11  | STORE_CMP_RES             | operand 1          | operand 2       | result       | `CmpType`  | Push bool result of comparison to stack
| 12  | SET_FLAGS_CMP_WITH_ZERO   | operand            |                 |              |            | Compare with zero and set comparison flags
| 13  | MATH_OPER                 | operand 1          | operand 2       | result       | `MathOper` | Math operation
| 14  | JUMP                      |                    |                 | dest block i | `JmpType`  | Conditional or unconditional jump
| 15  | READ_DOUBLE               |                    |                 | value        |            | Read double precision floating point number from user
| 16  | PRINT_DOUBLE              | value              |                 |              |            | Print double precision floating point number
| 17  | SET_FPS                   | value              |                 |              |            | SPU: asm `fps <value>` - set max fps count for video mode
| 18  | SHOW_VIDEO_FRAME          |                    |                 |              |            | SPU: asm `shw` - show image frame in video mode

### Формат `IRVal`

```
struct IRVal {
    enum {
        NONE  = 0,

        CONST      = 1,
        INT_CONST  = 2,
        LOCAL_VAR  = 3,
        GLOBAL_VAR = 4,
        ARG_VAR    = 5,
        ARR_VAR    = 6,
        STK        = 7,
        REG        = 8,
        ADDR       = 9,
    } type = NONE;

    union {
        size_t offset = 0; //< memory offset (for variables)
        double k_double;   //< const double number
        long   k_int;      //< const integer number
        size_t reg;        //< register number
        size_t rsp;        //< internal compiler stack pointer
        size_t addr;       //< another block address (physical index in list)
    } num;
};
```

### Типы `CmpType`

| Num | Name          | Description |
|:---:|:--------------|:------------|
|  1  | GREATER       | `dest[0] = src[0] >  src[1]`
|  2  | LOWER         | `dest[0] = src[0] <  src[1]`
|  3  | NOT_EQUAL     | `dest[0] = src[0] != src[1]`
|  4  | EQUAL         | `dest[0] = src[0] == src[1]`
|  5  | GREATER_EQUAL | `dest[0] = src[0] >= src[1]`
|  6  | LOWER_EQUAL   | `dest[0] = src[0] <= src[1]`

### Типы `JmpType`

| Num | Name          | Description |
|:---:|:--------------|:------------|
|  1  | UNCONDITIONAL | `jmp dest[0]`
|  2  | IS_ZERO       | `jz dest[0]`
|  3  | IS_NOT_ZERO   | `jnz dest[0]`

### Типы `MathOper`

| Num | Name | Description |
|:---:|:-----|:------------|
|  1  | ADD  | `dest[0] = src[0] + src[1]`
|  2  | SUB  | `dest[0] = src[0] - src[1]`
|  3  | MUL  | `dest[0] = src[0] * src[1]`
|  4  | DIV  | `dest[0] = src[0] / src[1]`
|  5  | POW  | `dest[0] = pow(src[0], src[1])`
|  6  | SQRT | `dest[0] = sqrt(src[0])`
|  7  | SIN  | `dest[0] = sin(src[0])`
|  8  | COS  | `dest[0] = cos(src[0])`
|  9  | LN   | `dest[0] = log(src[0])`
| 10  | NEG  | `dest[0] = -src[0]`
