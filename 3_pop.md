# Протоколо-ориентированное программирование

## Что такое протокол (protocol)?
Протокол описывает методы и свойства, которые должны реализовать объекты реализующие этот протокол. Методы описанные в протоколе не имеют собственной реализации. Они реализуются в реализующем протокол классе или структуре. Протокол - это интерфейс, то есть он содержит абстрактные методы, константы и переменные.

### Что такое категория (category) и когда она используется?
Механизм, позволяющий расширять уже существующие классы без применения наследования. Категория имеет доступ к переменным экземпляра исходного класса, однако с помощью категорий невозможно добавить новые переменные экземпляра. В Swift это называется расширением (extension).

### Что такое протоколы для класса? Зачем их использовать?
Можно ограничть принятие протокола типами классов (а не структурами или перечислениями), добавив протокол class в список наследования протокола. В таком случае свойства, в которые передается значение типа этого протоколоа может быть со слабой ссылкой (weak).

### Как создать опциональный метод протокола в Swift?
В Swift нету опциональных методов протокала, таких как в Objective-C (`optional`). Но вместо опциональных методов мы можем использовать значения по умолчанию.
```swift
protocol Countable {
  func getCount() -> Int
}

extension Countable {
  func getCount() -> Int {
    // Значение, которое будет использоваться по умолчанию
    return 0
  }
}

struct EmptyMarket: Countable {
  // Мы реализуем протокол, но не объявляем метод `getCount()`
}

struct Market: Countable {
  // Мы реализуем протокол и объявляем метод `getCount()`  
  let productsCount: Int

  func getCount() -> Int {
    return self.productsCount
  }
}

let emptyMarket = EmptyMarket()
emptyMarket.getCount() // 0

let market = Market(productsCount: 25)
market.getCount() // 25
```

### Как создать хранимое свойство в расширении к протокола?
Для этого нужен протокол для класса и умение грамотно использовать `objc_getAssociatedObject()` и `objc_setAssociatedObject()`. Например:
```swift
protocol Countable: class {
  var count: Int { set get }
}

fileprivate var countableCountKey = "Countable.count"
extension Countable {
  var count: Int {
    set {
        objc_setAssociatedObject(self, &countableCountKey, newValue, .OBJC_ASSOCIATION_ASSIGN)
    }
    get {
        return objc_getAssociatedObject(self, &countableCountKey) as? Int ?? 0
    }
  }   
}

class Market: Countable {

}

let market = Market()
market.count // 0
market.count = 17
market.count // 17
```

### Как проверять, реализован ли протокол в определнном классе или структуре?
Для этого можно использовать операторы `is` для проверки соответствия протоколу и `as` для приведения к определенному протоколу.
```swift
protocol Countable {
  var count: Int { get }
}

extension Int: Countable {
  var count: Int {
    return self
  }
}

let array: [Any] = [5, 3.5, "25"]

let filteredArray: [Any] = array.filter { $0 is Countable }
filteredArray // [5] - Но тип элемента массива так и остался Any

let countableArray: [Int] = array.compactMap { $0 as? Countable }
countableArray // [5] - Тип элемента массива изменился на Int
```

## Что такое протоколо-ориентированное программирование?
Это парадигма дизайна, которая позволяет группировать похожие методы, функции и свойства применительно к классам, структурам и перечислениям. При том как в ООП,  только классы позволяют использовать наследование от базового класса.

### В чем преимущества протоколо-ориентированного программирования?
ПОП может не только заменить наследнование, но и обеспечить совместную функциональность между классами.
- Можно реализовать несколько протоколов одним классов, а наследовать можно только один класс
- Можно реализовать протокол не только для класса, но и для структуры или перечислений 
- Нет обязательного условия наследовать все классы, можно реализовать только определенные протоколы

## Разница между NSCoding и Codable?
`NSCoding` используется для поддержки кодирования и декодирования экземпляров в iOS класс должен принять протокол NSCoding и реализовать его методы `init(coder:)` и `encode(with:)`, которые должны содержать код для каждого свойства, которое необходимо закодировать или декодировать:
- `init(coder:)` возвращает объект, инициализированный из данных в данном `unarchiver`
- `encode(with:)` кодирует получатель с помощью заданного архиватора.

`Codable` делает ваши типы данных кодируемыми и декодируемыми для совместимости с внешними представлениями, такими как JSON. Он также обеспечивает поддержку классов, структур и перечислений.