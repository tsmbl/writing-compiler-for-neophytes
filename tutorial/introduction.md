# Теория компиляторов для неофитов

## Введение

[Как построен этот курс <](howto.md)
[Содержание](content.md)
[> Устройство компилятора](phases.md)

-------

Компьютеры могут показывать кино, играть музыку, отрисовывать неотличимые от
фотографии изображения.  Могут рисовать красивые шрифты, подготовить книгу к
печати, рисовать формулы красивее человека.  Могут считать интегралы и играть в
шахматы.  Компьютеры могут почти все что угодно.  Однако кто из нас знает, как
компьютер делает все вышеперечисленное?

Для многих из нас компьютер сродни магическому шару, который показывает
картинки по воле высших сил.  Разработавшие компьютеры инженеры знают, что
компьютер состоит из большого числа относительно простых узлов, каждый из
которых делает простые операции, вроде добавить к значению единицу, или
скопировать значение из одной ячейки в другую.  Хотя инженеры знают, как
компьютер сделан, для них также тяжело вообразить, как с помощью простейших
арифметических операций сделать браузер, показывающих динамические
веб-страницы.  С подобной ситуацией люди сталкивались, когда делали
механические арифмометры: как проводить арифметические операции людям хорошо
известно, однако, чтобы выразить эти операции на языке шестеренок нужны
нетривиальные усилия.

Чтобы облегчить труд программиста, были созданы языки программирования, которые
выражали высокоуровневые конструкции и концепции в терминах операций над
аппаратным обеспечением.  Развитие языков программирования шло постепенно.
Сначала был создан ассемблер, позволивший записывать программный код в виде
понятных человеку команд, а не в виде шестнадцатеричных кодов.  Однако
ассемблер по прежнему обязывал человека оперировать на уровне регистров
процессора, операций ввода-вывода и т.п., поэтому написание даже самой короткой
программы занимало немало времени.

Язык C представляет собой пример абстракции более высокого уровня, чем
ассемблер, так он позволяет работать с циклами, именованными локальными
переменными, функциями с различными числом аргументов, определять сложные типы
данных.  Использование C также позволило программистом легче переиспользовать
написанный ранее код, так как код на C не зависел от архитектуры машины, на
которой должна была исполняться программа, в то время, как ассемблер был
специфичен для каждого процессора.  Однако, если программист оказался избавлен
от задачи перевода исходного кода в машинные инструкции, то кто-то или что-то
должно было этим заняться.  Эта задача легла на виртуальные плечи компилятора.

Формально компилятор можно определить как программное средство, выполняющую
трансляцию исходного кода высокого уровня в более элементарный код, чаще всего
в машинный код.  Понятие высокоуровнего и низкоуровнего кода тяжело определить,
однако можно перечислить следующие характеристики, специфичные для каждого
уровня кода.

Низкоуровневый код:

1. Учитывает аппаратные особенности машины, например,
   наличие и количество регистров, наличие стека, виды доступа к памяти и т.п.
1. Содержит инструкции реализованные аппаратно или тривиально выражаемые через
   элементарные.
1. Позволяет контролировать все аспекты выполняемой программы.

Высокоуровневый код:

1. Содержит универсальные команды, выполняемые на процессорах самых разных
   архитектур
1. Позволяет определять любое количество именованных переменных.
1. Стандартизирует работу со стеком, задает стандарт вызова функций.
1. Упорядочивает работу с памятью, например, позволяет использовать кучу
   (heap).
1. Позволяет работать со сложными типами данных, такими как массивы, структуры, 
   перечисления и т.п.

Для иллюстрации рассмотрим небольшую программу на C, вычисляющую сумму
арифметической прогрессии: **test.c**

```c
#include <stdio.h>
void main() {
    const int N=100;
    int sum=0;
    int n;
    for(n=0;n<N;n++) sum+=n;
    printf("Sum = %d\n", sum);
}
```

Откомпилируем программу с помощью компилятора C из GNU Compiler Collection:

```bash
gcc test.c -S -o test.asm
```

В результате получаем следующий код ассемблера: **test.asm**

```asm
    .file "test.c"
    .section .rodata
.LC0:
    .string "Sum = %d\n"
    .text
    .globl  main
    .type   main, @function
main:
.LFB0:
    .cfi_startproc
    pushq   %rbp
    .cfi_def_cfa_offset 16
    .cfi_offset 6, -16
    movq    %rsp, %rbp
    .cfi_def_cfa_register 6
    subq    $16, %rsp
    movl    $100, -4(%rbp)
    movl    $0, -12(%rbp)
    movl    $0, -8(%rbp)
    jmp .L2
.L3:
    movl    -8(%rbp), %eax
    addl    %eax, -12(%rbp)
    addl    $1, -8(%rbp)
.L2:
    movl    -8(%rbp), %eax
    cmpl    -4(%rbp), %eax
    jl      .L3
    movl    -12(%rbp), %eax
    movl    %eax, %esi
    movl    $.LC0, %edi
    movl    $0, %eax
    call    printf
    nop
    leave
    .cfi_def_cfa 7, 8
    ret
    .cfi_endproc
.LFE0:
    .size    main, .-main
    .ident   "GCC: (Ubuntu 5.4.0-6ubuntu1~16.04.4) 5.4.0 20160609"
    .section .note.GNU-stack,"",@progbits
```

Ассемблерный код больше, его тяжелее читать, тяжелее изменять, неудивительно,
что разработка на ассемблере занимает гораздо больше времени, чем на C.  Однако
использование высокоуровневого языка имеет издержки, выражаемые в том, что
генерируемый машинный код больше написанного программистом, скомпилированная
программа может работать медленнее, написанной на ассемблере.

Дальнейшее развитие языков программирования позволило ускорить разработку
далее, добавив более абстрактные конструкции, такие как классы и объекты,
замыкания, дженерики и т.п.  Кроме ускорения разработки, развитие языков
программирования преследовало и другие цели, например, уменьшение шансов
сделать программистом ошибку, облегчение разработки в больших коллективах,
поддержка появляющихся новых аппаратных возможностей, улучшение переносимости
программ между архитектурами.  В результате прогресса в развитии языков
программирования, появились самые популярные на настоящий момент языки
программирования C++ и Java.

В настоящее время дальнейшее развитие языков программирования, делающее акцент
на функциональном программирование, широком использование параллелизма
вычислений, безопасной работой с памятью при сохранении скорости работы.  Таким
образом возникли, Haskell, Rust и др.

Следует отметить, что не все языки программирования являются компилируемыми,
т.е. код которых транслируется в машинный код.  Ряд языков, например, Python и
Forth, являются интерпретируемыми, т.е. исходный код на этих языках выполняется
программой интерпретатором, без преобразования в машинный код.  Скорость
выполнения интерпретируемого кода очевидно значительно меньше, чем скорость
компилируемого кода в связи с наличием большого числа издержек на
интерпретацию, включая разбор исходного кода, поиск переменных по имени,
приращение внутренних указателей и т.п.  Впрочем интерпретаторы часто
используют промежуточное представление кода, хранящее разобранное
синтаксическое дерево, чтобы избежать синтаксического анализа кода при
выполнении каждой инструкции.  Более того, для языка может существовать как
интерпретатор, так и компилятор.  В последнее время получил распространение
подход заключающийся в компиляции кода в памяти, непосредственно перед
выполнением, называемый JIT.

Компилятор не всегда транслирует исходный код в машинные инструкции, вместо
этого иногда используются инструкции некой виртуальной, реально не
существующей, машины, инструкции которой выполняются специальной программой,
также обычно называемой виртуальной машиной.  Код виртуальной машины называют
байткодом или промежуточным представлением.  Код виртуальной машины напоминает
вспомогательное представление программы, используемое интерпретаторами, однако
код виртуальной машины обычно более прост, содержит меньше разных инструкций,
что позволяет виртуальной машины исполнять его быстрее, чем это делает
интерпретатор.  Однако программа, работающая на виртуальной машине, очевидно
работает медленнее, чем программа на машинном коде.  Однако трансляция в
промежуточный код позволяет упростить написание компилятора, перенеся адаптацию
к конкретному аппаратному обеспечению на написание виртуальной машины, которая
значительно проще компилятора.  В результате с помощью одного компилятора
удается компилировать код под все архитектуры.  В настоящее время наиболее
известны следующие виртуальные машины:

* [Java Virtual Machine](https://ru.wikipedia.org/wiki/Java_Virtual_Machine)
  (сокращенно JVM), используется языками Clojure, Scala, Kotlin и др.
* [Common Language Infrastructure](https://ru.wikipedia.org/wiki/Common_Language_Infrastructure)
  (сокращенно CLI), используется языками C#, F#, Visual Basic .NET и др.

Со временем компиляторы стали объединяться в наборы компиляторов, чтобы
переиспользовать их общие код, например, оптимизацию генерируемого кода.  В
результате возникли следующие наборы, включающие компиляторы многих языков:

* [GCC, the GNU Compiler Collection](https://gcc.gnu.org/)
  компилирует языки C, C++, Fortran, Ada, Go и др.
* [The LLVM Compiler Infrastructure](http://llvm.org/)
  используется компиляторами CLang, Rust, Julia, CUDA и др.

В настоящее время даже если вы не занимаетесь разработкой программного
обеспечения, на вашем компьютере установлено несколько компиляторов, например:

* Драйвера видеокарт содержат компиляторы шейдеров GLSL и OpenCL.
* Ваш браузер содержит модуль исполнения JavaScript, который использует JIT
  компиляцию для ускорения выполнения скриптов на веб-страницах.
* Поиск по регулярному выражению в текстовом редакторе может компилировать
  регулярное выражение для быстродействия.


### Вопросы для закрепления

* В каком случае разработка на ассемблере предпочтительнее, чем на
  высокоуровневом языке?
* Почему виртуальные машины получили широкое распространение?
* Где проходит граница между интерпретатором и компилятором?

### Задания

* Ознакомиться с архитектурой и машинными инструкциями процессора x86.
* Ознакомиться с архитектурой виртуальной машины Java.
* Написать простую программу на Java или C, откомпилировать ее, изучить
  сгенерированный код.

### Список литературы

* [Инструкции процессора x86](http://www.felixcloutier.com/x86/)
* [Спецификация виртуальной машины Java](https://docs.oracle.com/javase/specs/jvms/se7/html/)

-------

[Как построен этот курс <](howto.md)
[Содержание](content.md)
[> Устройство компилятора](phases.md)
