# Многопоточность

## Что такое GCD (Grand central dispatch)?
Это технология Apple для управления несколькими потоками. GCD предоставляет и управляет очередями задач. Хороший пример - когда приложение извлекает данные из API, этот сетевой вызов должен выполняться в фоновом потоке, а отображение данных в представлении должно выполняться в основном потоке, а также любые обновления пользовательского интерфейса.

## Что такое очереди отправки (dispatch queue)?
Это способ асинхронного и синхронного выполнения задач в вашем приложении. Задача - это просто некоторая работа, которую необходимо выполнить вашему приложению. Например, вы можете определить задачу для выполнения некоторых вычислений, создания или изменения структуры данных, обработки некоторых данных, считанных из файла, или любого количества вещей. Вы определяете задачи, помещая соответствующий код в функцию или объект блока и добавляя его в очередь отправки.

### Какие типы очередей есть? Для каких задач лучше использовать каждый тип?
1. Последовательные очереди (serial - в Swift, DISPATCH_QUEUE_SERIAL - в Objective-C). Выполняют одну задачу за раз в том порядке, в котором они добавляются в очередь. Текущая выполняемая задача выполняется в отдельном потоке (который может варьироваться от задачи к задаче), который управляется очередью отправки. Последовательные очереди часто используются для синхронизации доступа к определенному ресурсу.
```swift
let serialQueue = DispatchQueue(label: "serial")

serialQueue.async {
  print("Задача 1")
  for number in (0...5) {
    print("Задача 1 - \(number)")
  }
  print("Задача 1 - выполнена")
}

serialQueue.async {
  print("Задача 2")
  for number in (0...2) {
    print("Задача 2 - \(number)")
  }
  print("Задача 2 - выполнена")
}

// Задача 1
// Задача 1 - 0
// Задача 1 - 1
// Задача 1 - 2
// Задача 1 - 3
// Задача 1 - 4
// Задача 1 - 5
// Задача 1 - выполнена
// Задача 2
// Задача 2 - 0
// Задача 2 - 1
// Задача 2 - 2
// Задача 2 - выполнена
```
2. Параллельные очереди (concurent - в Swift, DISPATCH_QUEUE_CONCURRENT - в Objective-C). Выполняют одну или несколько задач одновременно, но задачи по-прежнему запускаются в том порядке, в котором они были добавлены в очередь. Текущие выполняемые задачи выполняются в разных потоках, которые управляются очередью отправки. Точное количество задач, выполняемых в любой момент, варьируется и зависит от состояния системы.
В iOS 5 и более поздних версиях вы можете самостоятельно создавать параллельные очереди отправки, указав DISPATCH_QUEUE_CONCURRENT в качестве типа очереди. Кроме того, ваше приложение может использовать четыре предопределенных глобальных параллельных очереди. Для получения дополнительной информации о том, как получить глобальные параллельные очереди, см. Получение глобальных параллельных очередей отправки.
 (очередь отправки, которая выполняет блоки одновременно. Хотя они выполняют блоки одновременно, вы можете использовать барьерные блоки для создания точек синхронизации в очереди.)
```swift
let concurentQueue = DispatchQueue(label: "concurent", attributes: .concurrent)

concurentQueue.async {
  print("Задача 1")
  for number in (0...5) {
    print("Задача 1 - \(number)")
  }
  print("Задача 1 - выполнена")
}

concurentQueue.async {
  print("Задача 2")
  for number in (0...2) {
    print("Задача 2 - \(number)")
  }
  print("Задача 2 - выполнена")
}

// Задача 1
// Задача 2
// Задача 1 - 0
// Задача 2 - 0
// Задача 1 - 1
// Задача 1 - 2
// Задача 2 - 1
// Задача 1 - 3
// Задача 2 - 2
// Задача 1 - 4
// Задача 2 - выполнена
// Задача 1 - 5
// Задача 1 - выполнена
```
3. Основная очередь отправки (main) - это глобально доступная последовательная очередь, которая выполняет задачи в основном потоке приложения. Поскольку она выполняется в основном потоке вашего приложения, основная очередь часто используется в качестве ключевой точки синхронизации для приложения и используется для обновлений пользовательского интерфейса. Вы можете начать выполнять тяжелую работу в фоновой очереди и отправлять запросы обратно в основную очередь, когда закончите:
```swift
let concurrentQueue = DispatchQueue(label: "concurrent.queue", attributes: .concurrent)

concurrentQueue.async {
  // Выполните запрос для получения данных и декодируем JSON в фоновой очереди.
  fetchSomeNetworkAndDecodeData()
  // Переводим задачу в основной поток
  DispatchQueue.main.async {
    /// Обновляем пользовательский интерфейс в основной очереди.
    tableView.reloadData()
  }
}
```

### Каким способом можно сделать доступ к ресурсу потокобезопасным?
Использование барьера в параллельной очереди для синхронизации записи, сохраняя при этом возможность одновременного чтения. Вы можете рассматривать барьер как задачу, которая мешает параллельным задачам и на мгновение превращает параллельную очередь в последовательную. Задача, выполняемая с барьером, откладывается до тех пор, пока все ранее отправленные задачи не завершатся. После завершения последней задачи очередь выполняет блок-барьер и после этого возобновляет свое обычное поведение выполнения.

Следующий код демонстрирует класс мессенджера, к которому можно получить доступ из нескольких потоков одновременно. Добавление новых сообщений в массив выполняется с помощью флага барьера, который блокирует новые чтения до завершения записи.
```swift
final class Messenger {
  private var messages: [String] = []
  private var queue = DispatchQueue(label: "messages.queue", attributes: .concurrent)

  var lastMessage: String? {
    return queue.sync {
      messages.last
    }
  }

  func postMessage(_ newMessage: String) {
    queue.sync(flags: .barrier) {
      self.messages.append(newMessage)
    }
  }
}
```

### В чем разница между синхронной и асинхронной задачей?
При синхронном выполнении, функция выполняется и ожидает завершения задачи. Таким образом она блокирует текущий поток пока задача не будет завершена.

При асинхронном выполнении, функция немедленно возвращается, приказывая выполнить задачу, но не дожидаясь ее. Таким образом, асинхронная функция не блокирует текущий поток выполнения от перехода к следующей функции.

### Как узнать, что несколько параллельных потоков выполнили свою задачу?
1. Использовать `OperationQueue` для создание очереди, где каждая задача может быть зависима от другой:
```swift
let operationQueue = OperationQueue()

let blockOperation_1 = BlockOperation {
  print("Block: 1")
}

let blockOperation_2 = BlockOperation {
  print("Block: 2")
}
blockOperation_2.addDependency(blockOperation_1)

operationQueue.addOperations([blockOperation_1, blockOperation_2], waitUntilFinished: false)

blockOperation_2.completionBlock = {
  print("Completed")
}
```
2. Использовать `DispatchGroup` для создание счетчика асинхронных операций, где `enter()` увеличивает счетчик на 1, а `leave()` уменьшает его. Как только счетчик уменьшится до 0, выполнится блок уведомления `notify()`.
```swift
let dispatchGroup = DispatchGroup()

dispatchGroup.enter()
DispatchQueue.global().async {
  print("Block: 1")
  dispatchGroup.leave()
}

dispatchGroup.enter()
DispatchQueue.global().async {
  print("Block: 2")
  dispatchGroup.leave()
}


dispatchGroup.notify(queue: .main) {
  print("Completed")
}
```

### Что такое DispatchSemaphore? Зачем его использовать
Семафор позволяет выполнять какой-либо участок кода одновременно только конкретному количеству потоков. В основе семафора лежит счетчик, который и определяет, можно ли выполнять участок кода текущему потоку или нет. Если счетчик больше нуля — поток выполняет код, в противном случае — нет.
```swift
let dispatchGroup = DispatchGroup()
let dispatchSemaphore = DispatchSemaphore(value: 0)

dispatchGroup.enter()
DispatchQueue.global().async {
  print("Block 1 - wait")
  print("Block 1 - start")
  sleep(1)
  print("Block 1 - finished")
  dispatchSemaphore.signal()
  dispatchGroup.leave()
}

dispatchGroup.enter()
DispatchQueue.global().async {
  print("Block 2 - wait")
  dispatchSemaphore.wait()
  print("Block 2 - start")
  sleep(2)
  print("Block 2 - finished")
  dispatchSemaphore.signal()
  dispatchGroup.leave()
}

dispatchGroup.notify(queue: .main) {
  print("Completed")
}
```

### Что такое @synchronized?
Он гарантирует, что только один поток может выполнять код в блоке в любой момент времени. В Swift используется `DispatchQueue.sync`

### Что такое Deadlock?
Ситуация в многозадачной среде, при которой несколько процессов находятся в состоянии бесконечного ожидания ресурсов, захваченных самими этими процессами. Например если вызвать DispatchQueue.main.sync в другом DispatchQueue.main.sync
```swift
let serialQueue = DispatchQueue(label: "concurrent.queue")

serialQueue.sync {
  // ...
  serialQueue.sync { // <- Deadlock
    // ...
  }
}
```

### Что такое Livelock?
Livelock частая проблема в асинхронных системах. Потоки почти не блокируются на критических ресурсах. Вместо этого они выполняют свою небольшую не блокируемую задачу и отправляют её в очередь на обработку другими потоками. Может возникнуть ситуация, когда потоки друг другу начинают перекидывать какое-то событие и его обработка зацикливается. Явного бесконечного цикла, как бы, не происходит, но нагрузка на асинхронную систему резко возрастает. В результате чего эти потоки больше ничем не успевают занимаются.

## Что такое качества обслуживания (quality of service или сокращенно qos) очередей?
Блоки очередей, отправленные в глобальную очередь с высоким приоритетом, будут вызываться раньше, чем блоки очередей, отправленные в глобальные очереди по умолчанию или с низким приоритетом. Блоки, отправленные в глобальную очередь с низким приоритетом, будут вызываться только в том случае, если нет ожидающих блоков в очередях по умолчанию или в очередях с высоким приоритетом.
Таким образом, это очереди, которые запускаются в фоновых потоках, когда они становятся доступными. Они не относятся к системе FIFO, поэтому порядок их размещения не гарантируется.

### Какие виды качества обслуживания (quality of service или сокращенно qos) очередей вы знаете? Для чего каждую из них использовать?
1. `userInteractive` - Эта очередь имеет самый высокий приоритет, но ниже, чем у main. Используется для задач, которые взаимодействуют с пользователем в данный момент и занимают очень мало времени. Например, когда пользователь возит пальцем и нам нужно провести определнные расчеты.
2. `userInitiated` - Эта очередь имеет высокий приоритет, но ниже, чем у предыдущей очереди. Используется для заданий, которые инициируются пользователем и требуют обратной связи. Например, когда пользователь выполнил определенное действие и ждет обратной связи, чтобы продолжить взаимодействие. Действие может занять несколько секунд.
3. `utulity` - Эта очередь имеет нормальный приоритет. Используется для заданий, которые требуют некоторого времени для выполнения и не требуют немедленной обратной связи. Например, загрузка данных или очистка некоторой базы данных. Делается что-то, о чем пользователь не просит, но это необходимо для данного приложения. Задание может занять от несколько секунд до нескольких минут.
4. `background` - Эта очередь с низким приоритетом. Используется для заданий, не связанных с визуализацией и не критичных ко времени исполнения. Например синхронизация с web—сервисом. Обычно запускается в фоновом режиме, которая занимает значительное время от минут до часов.
5. `default` - Эта очередь находится между `userInitiated` и `utulity`.