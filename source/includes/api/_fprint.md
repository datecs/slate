# FPrint DSL

## Protocol

File or array list of commands. Every line is a single command.

`[command],[logical],[service];[semicolon separated arguments]`

where:

- [command] is current command letter or digit (see [reference](#reference))
- [logical] is logical number between 0..99
- [service] field holds command result data as follows:

    `[factory],[sequence],[command result]`
    `______,_,__` - not executed command

    In this field FPrint will return execution result of command.

    - [factory] is factory number of device which executed current command
    - [sequence] is command sequence which increments. Can be 0..9 and increments only on successfully executed command
    - [command result] execution code, can be 0 or some negative number. Check [errors codes](#errors)


### Java usage

> FPrintInterpreter init and execute commands

```java

private static final String FPRINT_EXAMPLE_SCRIPT =
            "48,1,______,_,__;1;0000;1234;0;\n" +
                    "S,1,______,_,__;Potatos;0.02;1.000;1;1;2;0;0;\n" +
                    "C,1,______,_,__;2;0.01;;;;\n" +
                    "S,1,______,_,__;Printer;0.05;3.000;1;1;2;0;0;\n" +
                    "T,1,______,_,__;4;;;;;\n" +
                    "C,1,______,_,__;1;10.00;;;;\n" +
                    "P,1,______,_,__; ;;;;;\n" +
                    "P,1,______,_,__;Is this fiscal printer;;;;;\n" +
                    "P,1,______,_,__;or is ;;;;;\n" +
                    "P,1,______,_,__;cash register?;;;;;\n" +
                    "P,1,______,_,__; ;;;;;\n" +
                    "T,1,______,_,__;\n" +
                    "D,1,______,_,__;\n" +
                    "";

final FiscalDevice device...;
final FPrintSettings s = new FPrintSettings.Builder().doCheck(true).build();
final FPrintInterpreter interpreter = FPrintInterpreter.create(device, s);

final ArrayList<String> list = new ArrayList<>(Arrays.asList(FPRINT_EXAMPLE_SCRIPT.split("\n")));

final List<FPrintCommand> result = interpreter.executeCommands(list);

```

> Work with FPrintCommand

```java

// command is already executed command
FPrintCommand command...;

if (command.getResult() != 0) {

  // Get error message
  System.out.println(command.getErrorMessage());

  // Or using static context
  System.our.println(FPrintCommand.getErrorMessage(command.getResult()));

  throw new FiscalException(command.getResult());
}

```

Datecs Fiscal Framework have out of the box FPrint support.

The only one thing you will need to do is to wrap your connected device with
`FPrintInterpreter`. Then you can execute list of strings representing commands.
Result will be `List<FPrintCommand>` which provides handy methods for check command
arguments or result.

You can check command result using `getResult()` on `FPrintCommand`. There are also getters for all command fields.

## Reference

### Open fiscal

`48,[logical],______,_,__;[operator];[password];[till number];[invoice]`


> Example send/receive of open fiscal command

```
> 48,1,______,_,__;1;1234;1;0
< 48,1,112233,1,0 ;1;1234;1;0
```

| Parameter       | Description                      | Values             |
| --------------- | -------------------------------- | ------------------ |
| `[operator]`    | Operator's name                  | 1..30 depends on device |
| `[password]`    | Operator's password              | 4..6 digits |
| `[till number]` | Operator's till number           | 1..65535 |
| `[invoice]`     | Indicate if invoice must be open | 1 for open invoice, 0 otherwise |

### Sale

`S,[logical],______,_,__;[item];[price];[quantity];[till number];[stock group];[VAT group];[invoice]`

> Example send/receive of sale command

```
> S,1,______,_,__;Potatos;0.02;1.000;1;1;2;0;0;
< S,1,112233,1,0 ;Potatos;0.02;1.000;1;1;2;0;0;
```

> Example of sale with adjustment

```
> S,1,______,_,__;Potatos;0.02;1.000;1;1;2;0;0;
> C,1,______,_,__;2;0.01;

< S,1,112233,1,0 ;Potatos;0.02;1.000;1;1;2;0;0;
< C,1,112233,2,0 ;2;0.01;

```

| Parameter       | Description                      | Values             |
| --------------- | -------------------------------- | ------------------ |
| `[item]`        | Item's name                      | up to 30 symbols   |
| `[price]`       | Item's single price              | -999999.99 to 999999.99 |
| `[quantity]`    | Item quantity                    | 0 to 99999.999     |
| `[till number]` | Operator's till number           | 1..65535           |
| `[stock group]` | Stock group                      | 1..99              |
| `[VAT group]`   | Stock group                      | 1..99              |
| `[invoice]`     | Indicate if invoice must be open | 1 for open invoice, 0 otherwise |

<aside class="notice">Note that command will open fiscal receipt on operator 1 and till number 1 if there's no already opened. </aside>

### Total

`T,[logical],______,_,__;` - All in cash or

`T,[logical],______,_,__;[type];[price];`


> Example send/receive of total command

```
> T,1,______,_,__;0;12.50;
< T,1,112233,1,0 ;0;12.50;
```

| Type | Description |
| ---- | ----------- |
| 0    | cash        |
| 1    | credit card |
| 2    | debit card  |
| 3    | check       |
| 4    | [subtotal](#subtotal) |
| 5..8 | custom payments |

<aside class="notice">You can execute total or print text until everything is not paid. After that fiscal receipt will be closed. </aside>

### Subtotal

Total with type `4`. This command must be followed by adjustment command.

> Subtotal with adjustment

```
> T,1,______,_,__;4;
> C,1,______,_,__;1;10.00;

< T,1,112233,1,0 ;4;
< C,1,112233,2,0 ;1;10.00;
```

### Adjustment

`C,[logical],______,_,__;[type][value]`

| Type | Description             |
| ---- | ----------------------- |
| 0    | Surcharge by percentage |
| 1    | Discount by percentage  |
| 2    | Surcharge by value      |
| 3    | Discount by value       |

<aside class="success">This command can be combined with sell or subtotal.</aside>
<aside class="warning">Using adjustment by value, value cannot be bigger than sale price </aside>

### Print text

Prints fiscal or non-fiscal text depending on opened receipt.

`P,[logical],______,_,__;[line 1];[line 2];[line 3];[line 4];[line 5];`

> Example usage or priting text

```
> P,1,______,_,__;In non-fiscal receipt;
> P,1,______,_,__;can execute;
> P,1,______,_,__;only command P;
> P,1,______,_,__;(for printing text) and;
> P,1,______,_,__;command T;
> P,1,______,_,__;for close of non-fiscal;
> P,1,______,_,__;receipt.;

< P,1,112233,5,0 ;In non-fiscal receipt;
< P,1,112233,6,0 ;can execute;
< P,1,112233,7,0 ;only command P;
< P,1,112233,8,0 ;(for printing text) and;
< P,1,112233,9,0 ;command T;
< P,1,112233,0,0 ;for close of non-fiscal;
< P,1,112233,1,0 ;receipt.;
```

### Service cash in and out

`I,[logical],______,_,__;[type];[price];`

| Type | Description |
| ---- | ----------- |
| 0    | Service in  |
| 1    | Service out |

> Example usage of service cash in/out

```
> I,1,______,_,__;0;1.5;;;;
> I,1,______,_,__;1;1.5;;;;

< I,1,424242,9,0 ;0;1.5;;;;
< I,1,424242,0,0 ;1;1.5;;;;
```

### Open non-fiscal receipt

`Y,[logical],______,_,__;`

<aside class="notice">In opened non-fiscal receipt only P (for print) and T (for close receipt) are allowed.</aside>

### Reports

`Z,[logical],______,_,__;[type];[year];[month]`

> Example usage of reports

```
> Z,1,______,_,__;0;
> Z,1,______,_,__;2;11;04;
> Z,1,______,_,__;3;11;03;
```

| Type | Year        | Month       | Description |
| ---- | ----------- | ----------- | ----------- |
| 0    |             |             | X report |
| 1    |             |             | Z report |
| 2    | format `YY` | format `MM` | Print short report for this date range |
| 3    | format `YY` | format `MM` | Print extended report for this date range |


### Print duplicate

`D,[logical],______,_,__;`


### Open drawer

`O,[logical],______,_,__;`

<aside class="warning"> This command will be executed only if device has drawer support.</aside>

### Control display

`L,[logical],______,_,__;[command];[type];[text];`

| Command | Type | Description |
| ------- | ---- | ----------- |
| 0       |      | Clear display |
| 1       |      | Clear first line |
| 2       |      | Clear second line |
| 3       | 0    | Display static text on first line |
| 3       | 1    | Display floating text on first line |
| 4       | 0    | Display static text on second line |
| 4       | 1    | Display floating text on second line |
| 5       |      | Display text on both first and second line |
| 6       |      | Display date time on first line |

<aside class="warning"> This command will be executed only if device has
external display support.</aside>

### Barcode

`A,[logical],______,_,__;[type];[value];`

| Barcode | Type | Value length |
| ------- | ---- | ------------ |
| EAN8    | 1    | 7 digits     |
| EAN13   | 2    | 12 digits    |
| Code128 | 3    | 9..18 symbols|
| ITF     | 4..5 | 2..5 digits  |

<aside class="warning">Barcode can be printed only in a fiscal or non-fiscal receipt.</aside>

### Cancel current receipt

`X,[logical],______,_,__;`
