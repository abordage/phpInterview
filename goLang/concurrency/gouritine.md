## Goroutines

Горутина — функция, которая может работать параллельно с другими функциями. Для создания горутины используется ключевое слово `go`, за которым следует вызов функции. Горутины очень легкие(примерно 4.5кб на горутину против нескольких мегобайт на поток POSIX).

## Отличия горутин от потоков

- Каждый поток операционной системы имеет блок памяти фиксированного разме­ра (зачастую до 2 Мбайт) для стека — рабочей области, в которой он хранит локальные переменные вызовов функций, находящиеся в работе или приостановленные на время вызова другой функции. В противоположность этому go-подпрограмма начинает работу с небольшим стеком, обычно около **2 Кбайт**. Стек go-подпрограммы, подобно стеку потока опе­рационной системы, хранит локальные переменные активных и приостановленных функций, но, в отличие от потоков операционной системы, не является фиксирован­ным; при необходимости он может расти и уменьшаться. Максимальный размер стека
  go-подпрограммы может быть около 1 Гбайта, на порядки больше типичного стека с фиксированным размером, хотя, конечно, такой большой стек могут использовать только несколько go-подпрограмм.
- Потоки операционной системы планируются в ее ядре, а  у go есть собственный **планировщик (m:n) мультиплексирующий(раскидывающий) горутинки(m) по потокам(n)**. Основной плюс = отсуствие оверхеда на переключение контекста.
- Планировщик Go использует параметр с именем **GOMAXPROCS** для определения, сколько потоков операционной системы могут одновременно активно выполнять код Go. Его значение по умолчанию равно количеству процессоров компьютера, так что на машине с 8 процессорами планировщик будет планировать код Go для выполне­ния на 8 потоках одновременно (**GOMAXPROCS** равно значению п в т:п-планировании).
  Спящие или заблокированные в процессе коммуникации go-подпрограммы потоков для себя не требуют. Go-подпрограммы, заблокированные в операции ввода-вывода или в других системных вызовах, или при вызове функций, не являющихся функция­ми Go, нуждаются в потоке операционной системы, но **GOMAXPROCS** их не учитывает.
- В большинстве операционных систем и языков программирования, поддержива­ющих многопоточность, текущий поток имеет идентификацию, которая может быть легко получена как обычное значение (обычно — целое число или указатель). Это облегчает построение абстракции, именуемой локальной памятью потока, которая, по существу, является глобальным отображением, использующим в качестве ключа идентификатор потока, так что каждый поток может сохранять и извлекать значения независимо от других потоков.
  **У горутин нет идентификации, доступной программисту.** Так решено во время проектирования языка, поскольку локальной памятью потока про­граммисты злоупотребляют.

## context.Context

Хорошей практикой, считается "управлять" горутинами через контекст:

```go
// Контекст предоставляет механизм дедлайнов, сигнал отмены, и доступ к запросозависимым значениям.
// Эти методы безопасны для одновременного использования в разных go-рутинах.
type Context interface {
    // Done возвращает канал, который закрывается когда Context отменяется
    // или по таймауту.
    Done() <-chan struct{}

    // Err объясняет почему контекст был отменен, после того как закрылся канал Done.
    Err() error

    // Deadline возвращает время когда этот Context будет отменен.
    Deadline() (deadline time.Time, ok bool)

    // Value возвращает значение ассоциированное с ключем или nil.
    Value(key interface{}) interface{}
}
```



### Лучшие практики

1. **context.Background** следует использовать только на самом высоком уровне, как корень всех производных контекстов.
2. **context.TODO** должен использоваться, когда вы не уверены, что использовать, или если текущая функция будет использовать контекст в будущем.
3. Отмены контекста рекомендуются, но эти функции могут занимать время, чтобы выполнить очистку и выход.
4. **context.Value** следует использовать как можно реже, и его нельзя применять для передачи необязательных параметров. Это делает API непонятным и может привести к ошибкам. Такие значения должны передаваться как аргументы.
5. Не храните контексты в структуре, передавайте их явно в функциях, предпочтительно в качестве первого аргумента.
6. Никогда не передавайте nil-контекст в качестве аргумента. Если сомневаетесь, используйте TODO.
7. Структура `Context` не имеет метода cancel, потому что только функция, которая порождает контекст, должна его отменять.



*Дополнительно:*

- https://habr.com/ru/company/nixys/blog/461723/