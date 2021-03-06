# [Как работает пакет flag в Go](https://www.8host.com/blog/kak-rabotaet-paket-flag-v-go/ "Как работает пакет flag в Go")

20 декабря, 2019 11:47 дп 639 views | Комментариев нет

[Development](https://www.8host.com/blog/category/development/) | [Amber](https://www.8host.com/blog/author/amber/ "Записи Amber") | [1 Comment](https://www.8host.com/blog/kak-rabotaet-paket-flag-v-go/#disqus_thread)

Утилиты командной строки чаще всего нуждаются в дополнительной настройке. Хорошие стандартные параметры важны, но утилиты должны также принимать настройки от пользователей. На большинстве платформ утилиты принимают флаги, которые меняют стандартное поведение команды. Флаги — это ограниченные строки “ключ\-значение”, добавляемые после имени команды. Go позволяет создавать утилиты командной строки, которые принимают флаги через пакет flag из стандартной библиотеки.

В этом мануале вы научитесь использовать пакет flag для создания различных утилит командной строки. Мы покажем, как применять flag для управления выводом программы, вводить позиционные аргументы, в которых можно смешивать флаги и другие данные, а также реализовывать подкоманды.

## Изменение поведения программы с помощью пакета flag

Использование пакета flag состоит из трех этапов: сначала нужно определить переменные для захвата значений, затем определить флаги, которые будет использовать ваше приложение Go, и, наконец, обработать флаги, предоставленные приложению после выполнения. Большинство функций в пакете flag связаны с определением флагов и связыванием их с переменными, которые вы объявили. Этап обработки выполняется функцией Parse().

**Читайте также**: [Переменные и константы в Go](https://www.8host.com/blog/peremennye-i-konstanty-v-go/)

Для примера мы создадим программу, которая определяет логический флаг. Этот флаг будет менять сообщение, которое выводится на стандартный вывод. Если в команде есть флаг \-color, программа выведет сообщение синим цветом. Если флага нет, сообщение будет выведено стандартным цветом.

**Читайте также**: [Основы работы с логическими данными в Go](https://www.8host.com/blog/osnovy-raboty-s-logicheskimi-dannymi-v-go/)

Создайте новый файл boolean.go:

`nano boolean.go`

Поместите в файл такой код, чтобы создать программу:

```go
package main
import (
"flag"
"fmt"
)
type Color **string**
**const** (
ColorBlack  Color = "\u001b[30m"
ColorRed        = "\u001b[31m"
ColorGreen    = "\u001b[32m"
ColorYellow   = "\u001b[33m"
ColorBlue        = "\u001b[34m"
ColorReset    = "\u001b[0m"
)
func colorize(color Color, message **string**) {
fmt.Println(**string**(color), message, **string**(ColorReset))
}
func main() {
useColor := flag.Bool("color", **false**, "display colorized output")
flag.Parse()
**if** *useColor {
colorize(ColorBlue, "Hello, Darling!")
**return**
}
fmt.Println("Hello, Darling!")
}
```



В этом примере используются [управляющие последовательности ANSI](https://ru.wikipedia.org/wiki/%D0%A3%D0%BF%D1%80%D0%B0%D0%B2%D0%BB%D1%8F%D1%8E%D1%89%D0%B8%D0%B5_%D0%BF%D0%BE%D1%81%D0%BB%D0%B5%D0%B4%D0%BE%D0%B2%D0%B0%D1%82%D0%B5%D0%BB%D1%8C%D0%BD%D0%BE%D1%81%D1%82%D0%B8_ANSI). С их помощью терминал понимает, что нужно отображать цветной вывод. Поскольку это специализированные последовательности символов, имеет смысл определить для них новый тип. В этом примере мы назвали этот тип Color и определили его как строку. Затем мы определяем палитру цветов — это происходит в блоке const. Функция colorize, определенная после блока const, принимает одну из констант Color и строковую переменную с сообщением, которое нужно выделить цветом. Затем функция меняет цвет вывода в терминале: сначала отображается управляющая последовательность для необходимого цвета, после этого идет сообщение. В конце функция сбрасывает цвет терминала, выводя специальную последовательность для сброса цвета.

В main мы используем функцию flag.Bool, чтобы определить логический флаг color. Второй параметр этой функции, false, устанавливает значение по умолчанию для этого флага, если он не указан. Вопреки вашим ожиданиям, значение true не меняет поведение на обратное. Следовательно, с логическими флагами почти всегда используется значение false.

Последний параметр — это строка документации, которая может отображаться в качестве сообщения. Значение, возвращаемое из этой функции, является указателем на bool. Функция flag.Parse в следующей строке использует этот указатель для установки переменной bool на основе флагов, переданных пользователем. Затем мы можем проверить значение указателя bool, разыменовав его.

**Читайте также**: [Как работают указатели в Go](https://www.8host.com/blog/kak-rabotayut-ukazateli-v-go/)

Используя это логическое значение, мы можем вызвать colorize, если флаг \-color установлен, либо вызвать переменную fmt.Println, если флаг отсутствует.

Сохраните файл и запустите программу без флагов:

`go run boolean.go`

Вы получите такой вывод:

`Hello, Darling!`

Снова запустите программу, теперь с флагом \-color:

`go run boolean.go -color`

В выводе будет тот же текст, но на этот раз синим цветом.

Флаги — не единственные значения, которые можно передавать командам. Вы также можете отправить команде имена файлов или другие данные.

## Позиционные аргументы

Обычно команда принимает ряд аргументов, которые определяют предмет ее внимания. Например, команда head, которая выводит первые строки файла, часто вызывается как head example.txt. Файл example.txt является позиционным аргументом в вызове данной команды.

Функция Parse() будет продолжать анализировать все найденные флаги до тех пор, пока не обнаружит нефлаговый аргумент. Пакет flag использует для этого функции Args() и Arg().

Давайте для примера соберем упрощенную реализацию команды head, которая отображает несколько начальных строк заданного файла.

Для начала создайте файл head.go, а затем вставьте в него такие строки:

`package main
import (
"bufio"
"flag"
"fmt"
"io"
"os"
)
func main() {
**var** count **int**
flag.IntVar(&count, "n", 5, "number of lines to read from the file")
flag.Parse()
**var** **in** io.Reader
**if** filename := flag.Arg(0); filename != "" {
f, err := os.Open(filename)
**if** err != nil {
fmt.Println("error opening file: err:", err)
os.Exit(1)
}
defer f.Close()
**in** = f
} **else** {
**in** = os.Stdin
}
buf := bufio.NewScanner(**in**)
**for** i := 0; i < count; i++ {
**if** !buf.Scan() {
**break**
}
fmt.Println(buf.Text())
}
**if** err := buf.Err(); err != nil {
fmt.Fprintln(os.Stderr, "error reading: err:", err)
}
}`

Сначала мы определяем переменную count для хранения количества строк, которые программа должна вывести из файла. Затем мы определяем флаг \-n, используя flag.IntVar, что дублирует поведение исходной программы head. Эта функция (в отличие от функций flag, которые не имеют суффикса Var) позволяет передавать собственный указатель на переменную. Это единственное отличие, в остальном же параметры flag.IntVar аналогичны flag.Int: имя флага, значение по умолчанию и описание. Как и в предыдущем примере, мы затем вызываем flag.Parse() для обработки ввода пользователя.

Следующий раздел кода читает файл. Сначала мы определяем переменную io.Reader, которая будет установлена либо ​​в файл, запрошенный пользователем, либо в стандартный ввод, переданный программе. В операторе if мы используем функцию flag.Arg для доступа к первому позиционному аргументу, который идет после всех флагов. Если пользователь указал имя файла, оно будет установлено. В противном случае это будет пустая строка («»). Когда имя файла указано, мы используем функцию os.Open, чтобы открыть этот файл и установить для него переменную io.Reader, которую определили ранее. В противном случае мы используем os.Stdin для чтения из стандартного ввода.

В последнем разделе используется \*bufio.Scanner (созданный с помощью bufio.NewScanner) для чтения строк из переменной io.Reader. Мы выполняем итерацию до значения count с помощью цикла for, вызывая break, если при сканировании строки с помощью buf.Scan получается значение false — это значит, что количество строк меньше числа, запрошенного пользователем.

**Читайте также**: [Циклы For в Go](https://www.8host.com/blog/cikly-for-v-go/)

Запустите эту программу и выведите на экран содержимое файла, который вы только что написали, используя head.go в качестве аргумента файла:

`go run head.go -- head.go`

Разделитель — является специальным флагом, который значит для пакета flag, что далее флаговых аргументов нет. Вы получите такой вывод:

`package main
import (
"bufio"
"flag"`

Используйте флаг \-n, который вы определили ранее, чтобы откорректировать объем вывода:

`go run head.go -n 1 head.go`

Это выведет только оператор package:

`package main`

Наконец, когда программа обнаруживает, что позиционные аргументы не были предоставлены, она считывает ввод со стандартного ввода, как head. Попробуйте запустить эту команду:

`echo "fish\nlobsters\nsharks\nminnows" | go run head.go -n 3`

Вы получите такой вывод:

`fish
lobsters
sharks`

Поведение функций flag до сих пор ограничивалось вызовом команды. Такое поведение не всегда необходимо (особенно если вы пишете инструмент командной строки, который поддерживает подкоманды).

## Использование подкоманд с помощью FlagSet

Современные приложения командной строки часто реализуют «подкоманды» для объединения набора инструментов внутри одной команды. Наиболее известным инструментом, использующим этот шаблон, является git. Например, в команде “git init” git — это команда, а init — подкоманда. Важной особенностью подкоманд является тот факт, что каждая подкоманда может иметь свою собственную коллекцию флагов.

Приложения Go могут поддерживать подкоманды со своим собственным набором флагов, используя тип flag.(\*FlagSet). Чтобы посмотреть, как это работает, создайте программу, которая использует команду с двумя подкомандами, у которых есть разные флаги.

Создайте файл subcommand.go и вставьте в него следующие строки:

`package main
import (
"errors"
"flag"
"fmt"
"os"
)
func NewGreetCommand() *GreetCommand {
gc := &GreetCommand{
fs: flag.NewFlagSet("greet", flag.ContinueOnError),
}
gc.fs.StringVar(&gc.name, "name", "World", "name of the person to be greeted")
**return** gc
}
type GreetCommand **struct** {
fs *flag.FlagSet
name **string**
}
func (g *GreetCommand) Name() **string** {
**return** g.fs.Name()
}
func (g *GreetCommand) Init(args []**string**) error {
**return** g.fs.Parse(args)
}
func (g *GreetCommand) Run() error {
fmt.Println("Hello", g.name, "!")
**return** nil
}
type Runner **interface** {
Init([]**string**) error
Run() error
Name() **string**
}
func root(args []**string**) error {
**if** len(args) < 1 {
**return** errors.New("You must pass a sub-command")
}
cmds := []Runner{
NewGreetCommand(),
}
subcommand := os.Args[1]
**for** _, cmd := range cmds {
**if** cmd.Name() == subcommand {
cmd.Init(os.Args[2:])
**return** cmd.Run()
}
}
**return** fmt.Errorf("Unknown subcommand: %s", subcommand)
}
func main() {
**if** err := root(os.Args[1:]); err != nil {
fmt.Println(err)
os.Exit(1)
}
}`

Эта программа разделена на несколько частей: функция main, функция root и отдельные функции для реализации подкоманды. Функция main обрабатывает ошибки, возвращаемые командами. Если какая\-либо функция возвращает ошибку, оператор if ее перехватит, выведет ошибку, после чего программа завершит работу с кодом состояния 1 (это значит, что в остальной части системы произошла ошибка). В рамках функции main мы передаем все аргументы, с которыми была вызвана программа, пользователю root. Мы удаляем первый аргумент, который является именем программы (в данном примере это ./subcommand), создавая срез с помощью os.Args.

Функция root определяет \[\]Runner, где будут определены все подкоманды. Runner — это интерфейс для подкоманд, который позволяет root извлекать имя подкоманды с помощью Name() и сравнивать его с содержимым переменной subcommand. Как только  при итерации переменной cmds будет найдена правильная подкоманда, мы инициализируем подкоманду с остальными аргументами и вызываем метод Run() этой команды.

Здесь мы определяем только одну подкоманду, хотя эта структура позволяет нам легко создавать и другие. GreetCommand создается с помощью NewGreetCommand, где мы создаем новый \*flag.FlagSet с помощью flag.NewFlagSet. flag.NewFlagSet принимает два аргумента: имя установленного флага и стратегию сообщения об ошибках обработки. К имени \*flag.FlagSet можно получить доступ с помощью метода flag.(\* FlagSet).Name. Мы используем его в методе (\*GreetCommand).Name(), чтобы имя подкоманды совпадало с именем, которое мы присвоили \* flag.FlagSet. NewGreetCommand определяет флаг \-name аналогично предыдущим примерам, но тут он вызывается как метод из поля \*flag.FlagSet в \*GreetCommand, gc.fs. Когда root вызывает метод Init() для \*GreetCommand, мы передаем аргументы, предоставленные методу Parse поля \*flag.FlagSet.

Нам будет проще увидеть подкоманды, когда мы соберем программу и затем запустим ее:

`go build subcommand.go`

Теперь запустите программу без аргументов:

`./subcommand`

Вы получите такой вывод:

`You must pass a sub-command`

Теперь запустите программу с подкомандой greet:

`./subcommand greet`

Вы получите такой вывод:

`Hello World !`

Теперь давайте добавим в команду флаг \-name:

`./subcommand greet -name Darling`

Вы увидите такой вывод:

`Hello Darling !`

Этот пример демонстрирует лишь базовые принципы, на основе которых вы можете собрать более сложное приложение командной строки в Go. FlagSets дают разработчикам больше контроля над процессом обработки флагов.
