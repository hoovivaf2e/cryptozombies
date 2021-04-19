# Beginner to Intermediate Smart Contracts

## Lesson 1 - Making the Zombie Factory
- `zombiefactory.sol` - 製作zombie

## Learn

### State Variables & Integers
State variables 將會被永久地保存在合約中，也就是說他們被寫入到以太坊區塊鏈中了。
State variables Example:

```js
contract Example {
  // State variables，這個變數將會被永久儲存在區塊鏈中
  uint myUnsignedInteger = 100;
}
```

* uint 資料型態是 unsigned integer 的意思，意思是「其值不能是負數」。在Solidity中，uint實際上是uint256的簡寫，是256 bits的正整數。

- - -

### Math Operations
在 Solidity 中也有數學運算，與其他程式語言相同：
加法: x + y
減法: x - y
乘法: x * y
除法: x / y
取餘數: x % y (例如：13 % 5 餘 3)
Solidity 還支持次方運算 (如：x 的 y 次方） 
Example:

```js
uint x = 5 ** 2; // equal to 5^2 = 25
```

- - -

### Structs
structs:

```js
struct Person {
  uint age;
  string name;
}
```

Structs 可以形成一個更複雜的資料型態，它有多個屬性。

- - -

### Arrays
在 Solidity 中有兩種陣列: 固定陣列(fixed arrays) 和 動態陣列(dynamic arrays):

```js
// 固定長度為2的固定陣列:
uint[2] fixedArray;
// 固定長度為5的string類型的固定陣列:
string[5] stringArray;
// 動態陣列，長度不固定，可以動態添加元素:
uint[] dynamicArray;
```

```js
Person[] people; // 這是動態陣列，可以不斷添加元素
```

可以定義陣列為公共(public)，Solidity 會自動創建 getter 方法。語法如下:

```js
Person[] public people;
```

- - -

### Function Declarations
在 Solidity 中，函式定義的語法如下:

```js
function eatHamburgers(string memory _name, uint _amount) public {

}
```

呼叫語法如下：

```js
eatHamburgers("vitalik", 100);
```

* 習慣上，函式裡的變數都是以 `_` 開頭(但沒有硬性規定)以區別全域變數。

- - -

### Working With Structs and Arrays
上個例子中的 Person 結構:

```js
struct Person {
  uint age;
  string name;
}
Person[] public people;
```

現在建立一個新的 Person 結構，然后把它加入到 people 的陣列中.

```js
// 建立一個新的Person:
Person satoshi = Person(172, "Satoshi");
// 將新建的satoshi添加到people陣列中:
people.push(satoshi);
```

也可以用一行程式碼更簡潔:

```js
people.push(Person(16, "Vitalik"));
```

* `array.push()` 是從陣列的`尾部`加入新元素，所以元素在陣列中的順序就是我們添加元素的順序，如:

```js
uint[] numbers;
numbers.push(5);
numbers.push(10);
numbers.push(15);
// numbers is now equal to [5, 10, 15]
```

- - -

### Private / Public Functions
Solidity 中定義的函式的屬性默認為公共(public)。這就意味著任何一方 (或其它合約) 都可以調用你合約里的函式。
顯然，不是什麼時候都需要這樣，而且這樣的合約容易受到攻擊。所以將自己的函式定義為私有(private)是一個好的習慣，只有在當你需要外部世界調用它時才將它設置為公共。

定義一個私有的函式：

```js
uint[] numbers;

function _addToArray(uint _number) private {
  numbers.push(_number);
}
```

* 在函式名字後面使用關鍵字`private`即可，只有這合約中的其它函式才能夠調用這個函式。
* 和函式的參數類似，私有函式的名字慣用 `_` 開頭。

- - -

### More on Functions
**Return Values 回傳值**

希望函式回傳一個值，如下定義：

```js
string greeting = "What's up dog";

function sayHello() public returns (string) {
  return greeting;
}
```

Solidity 里，函式的定義裡可包含回傳值的資料型態(如本例中的`string`)。

modifiers 修飾符
上面的範例中並没有改变 Solidity 裡的狀態，也就是說，它没有改變任何值或者寫入任何東西。
這種情況下可以把函式定義為`view`，表示它只能讀取數據，不能更改數據:

```js
function sayHello() public view returns (string) {}
```

`pure`修飾符，表示這個函式不訪問合約裡的任何資料，例如：

```js
function _multiply(uint a, uint b) private pure returns (uint) {
  return a * b;
}
```

* 這個函式沒有讀取合約裡的任何變數，它的回傳值完全取決於輸入的參數，這種情況下我们把函式定義為`pure`。

- - -

### Keccak256 and Typecasting
Ethereum 内部有一個雜湊函數`keccak256`，它用了`SHA3`版本。一個雜湊函數基本上就是把一個字串轉換為一個256-bit的16進位數字。字串的一個微小變化都會引起雜湊函數極大的變化。

這在 Ethereum 中有很多應用，但是現在只是用它造一个偽隨機數。

例子:
```js
keccak256(abi.encodePacked("aaaab"));
//6e91ec6b618bb462a4a6ee5aa2cb0e9cf30f7a052bb467b0ba58b8748c00d2e5

keccak256(abi.encodePacked("aaaac"));
//b1f078126895a1424524de5321b339ab00408010b7cf0e6ed451514981e58aa9
```
上述範例可以看出，輸入的字串只改變了一個字母，輸出就已經是天壤之別了。

* 上述範例的方法並不安全，請勿用在真實的區塊鏈應用中。

**Typecasting**
有時你需要變換資料型態。例如:

```js
uint8 a = 5;
uint b = 6;
uint8 c = a * b;            // 將會抛出錯誤，因為 a * b 回傳 uint，而不是 uint8
uint8 c = a * uint8(b);     // 需要將 b 轉換為 uint8 就可以了
```

- - -

### Events

`事件`是合約和區塊鏈互動的一種機制。你的前端應用可以`監聽`某些事件，當事件觸發時也可以做出反應。

例子:

```js
event IntegersAdded(uint x, uint y, uint result);

function add(uint _x, uint _y) public {
  uint result = _x + _y;
  //觸發事件，通知app
  IntegersAdded(_x, _y, result);
  return result;
}
```

你的 app 前端可以監聽這個事件。JavaScript 實現如下:

```js
YourContract.IntegersAdded(function(error, result) {
  // ... 做出反應 ...
})
```


