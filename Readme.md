1)Какой самый эффективный способ конкатенации строк?  
StringBuilder  

2)Что такое интерфейсы, как они применяются в Go?  
Интерфейсы — это инструменты для определения наборов действий и поведения. Интерфейсы — это в первую очередь контракты. Они позволяют объектам опираться на абстракции, а не фактические реализации других объектов. При этом для компоновки различных поведений можно группировать несколько интерфейсов. В общем смысле — это набор методов, представляющих стандартное поведение для различных типов данных.

Как устроен Duck-typing в Go?

Если это выглядит как утка, плавает как утка и крякает как утка, то это, вероятно, утка и есть.

Если структура содержит в себе все методы, что объявлены в интерфейсе, и их сигнатуры совпадают — она автоматически удовлетворяет интерфейс.

Такой подход позволяет полиморфно (полиморфизм — способность функции обрабатывать данные разных типов) работать с объектами, которые не связаны в иерархии наследования. Достаточно, чтобы все эти объекты поддерживали необходимый набор методов.  

3)Чем отличаются RWMutex от Mutex?    
RWMutex и Mutex в Go - это два типа блокировок для управления доступом к общим ресурсам. RWMutex позволяет многим горутинам читать данные одновременно, но блокирует запись. Mutex же блокирует доступ для чтения и записи одновременно.

4)Чем отличаются буферизированные и не буферизированные каналы?   
Чтение или запись данных в небуферизированный канал блокирует горутину и контроль передается свободной горутине. Буферизированный канал создается указанием размера буфера, в этом случае горутина не блокируется до тех пор, пока буфер не будет заполнен. В го буффер реализован массивом и двумя указателями sendx и recvx. 
Также в небуфферизированном канале данные передаются напрямую в стек горутины.
 
5)Какой размер у структуры struct{}{}?    
Пустая структура занимает 0 байт.

6)Есть ли в Go перегрузка методов или операторов?     
Отсутсвует

7)В какой последовательности будут выведены элементы map[int]int?     
Порядок в итерации по мапе не гарантирован. Но fmt.Println(сам отсартирует мапу по ключам)

8)В чем разница make и new?    
new() - Переменная представляет именованный объект в памяти. Язык Go также позволяет создавать безымянные объекты - они также размещаются в памяти, но не имеют имени как переменные. Для этого применяется функция new(type). В эту функцию передается тип, объект которого надо создать. Функция возвращает указатель на созданный объект:
make() - Функция make() — это специальная встроенная функция, которая используется для инициализации слайсов, мап и каналов. make() можно использовать только для инициализации слайсов, мап и каналов, и что, в отличие от функции new(), make() не возвращает указатель. Например make(map[int]int)

9)Сколько существует способов задать переменную типа slice или map?
slice - 6
```
slice := []string{}
slice := []string{"a", "b"}
var slice []string
slice := make([]string, 0)
slice := make([]string, 0, 7)
slice := new([]string)
```
map - 5 способов
```
var m map[string]string
m := map[string]string{}
m := make(map[string]string)
m := new(map[string]string)
m := make(map[string]string, 3)
```

10)Что выведет данная программа и почему?
```
func update(p *int) {
  b := 2
  p = &b
}

func main() {
  var (
     a = 1
     p = &a
  )
  fmt.Println(*p)
  update(p)
  fmt.Println(*p)
}
```
По-умолчанию в go аргументы в функцию передаются по значению. Если необходимо передать указатель, нужно это явно указать. Однако в go нет ссылок (не возможно создать две переменные с одним адресом). В примере выше, несмотря на то, что в функцию update передается указатель, переменная в главной функции не изменяется. Обе переменные p указывают на один и тот же адрес, но это две разные переменные, поэтому из функции update мы не можем обновить переменную p в главной функции.

программа выведет:   
1  
1  

11)Что выведет данная программа и почему?
```
func main() {
  wg := sync.WaitGroup{}
  for i := 0; i < 5; i++ {
     wg.Add(1)
     go func(wg sync.WaitGroup, i int) {
        fmt.Println(i)
        wg.Done()
     }(wg, i)
  }
  wg.Wait()
  fmt.Println("exit")
}
```
Будет дедлок. Так как мы передаем структуру WaitGroup в функцию по значению, она копируется, и вызов wg.Done() внутри горутины не изменит счетчик WaitGroup в главной функции. Следовательно, главная горутина навсегда зависнет на wg.Wait()

Для того, чтобы исправить эту ошибку, необходимо передавать в функцию структуру WaitGroup по указателю, либо не передавать его и использовать через замыкание.

12)Что выведет данная программа и почему?
```
func main() {
  n := 0
  if true {
     n := 1
     n++
  }
  fmt.Println(n)
}
```
Выведет 0, вся суть в области видимости.

13)Что выведет данная программа и почему?
```
func someAction(v []int8, b int8) {
  v[0] = 100
  v = append(v, b)
}

func main() {
  var a = []int8{1, 2, 3, 4, 5}
  someAction(a, 6)
  fmt.Println(a)
}
```
Программа выведет: [100 2 3 4 5]

Вся суть в аллокации нового массива, на который мы начнем ссылаться после того за appendим новый элемент. А первое действие v[0] = 100 сработает.

14)Что выведет данная программа и почему?
```
func main() {
  slice := []string{"a", "a"}

  func(slice []string) {
     slice = append(slice, "a")
     slice[0] = "b"
     slice[1] = "b"
     fmt.Print(slice)
  }(slice)
  fmt.Print(slice)
}
```
Программа выведет: [b b a][a a]

В данной программе анонимной функции передается через параметры слайс, то есть он копируется. Добавив новое значие в слайс, вышли за пределы capacity который был равен 2, это 
значит что слайс в анонимной функции теперь ссылается на новый массив, теперь изменение этого слайса не конснется переданного слайса. 
