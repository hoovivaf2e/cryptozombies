# Beginner to Intermediate Smart Contracts

## Lesson 2 - Zombies Attack Their Victims
- `zombiefactory.sol`
- `zombiefeeding.sol`

## Learn

### Mappings and Addresses

**Addresses**
以太坊區塊鏈由`account`(帳戶)組成，你可以把它想像成銀行帳戶。帳戶的幣別是`Ether`(在以太坊區塊鏈上使用的幣種），你可以和其他帳戶之間支付和接受以太幣，就像你的銀行帳戶可以轉帳到其他銀行帳戶一樣。

每個帳戶都有一個“**地址**”，你可以把它想像成銀行帳戶的號碼。這是帳戶獨一的識別碼，它看起來長這樣：

`0x0cA446255506E92DF41619C46F1d6df9Cc960183`

**Mappings**

在 Lesson 1 中，有`structs`和`arrays`。`Mappings`是另一種在 Solidity 中儲存有組織的資料的方法。

`Mappings`的定義：
```js
mapping (address => uint) public accountBalance;    //將客戶的帳戶餘額保存在 uint型別的變數中

mapping (uint => string) userIdToName;  //透過 userId 儲存/查詢使用者名稱
```
`Mappings`本質上是儲存和查詢資料所用的鍵-值對。在第一個mapping中，鍵是`address`，值是`uint`。在第二個mapping中，鍵是`uint`，值是`string`。

- - -

### Msg.sender

在 Solidity 中，有一些全域變數可以被所有函式調用。 其中一個就是`msg.sender`，它指的是當前調用者（或智能合約）的`address`。

* 在 Solidity 中，執行函式始終是從外部調用開始。單單一個智能合約在區塊鏈上什麼也做不了，除非有人調用其中的函式，所以`msg.sender`總是需要的。

**msg.sender**

以下是使用`msg.sender`來更新`mapping`的例子：

```js
mapping (address => uint) favoriteNumber;

function setMyNumber(uint _myNumber) public {
  // 更新 `favoriteNumber` mapping 將 `_myNumber`儲存在 `msg.sender`下
  favoriteNumber[msg.sender] = _myNumber;
  // 將資料儲存到mapping的語法 跟 將資料儲存在陣列的語法 類似
}

function whatIsMyNumber() public view returns (uint) {
  // 拿到儲存在調用者地址名下的值
  // 若調用者還没調用 setMyNumber，則值為 `0`
  return favoriteNumber[msg.sender];
}
```

* 在上述例子中，任何人都可以調用`setMyNumber`，並在我們的合約中儲存一個`uint`，綁定他們的地址。然後，當他們調用`whatIsMyNumber`時，就會得到他們儲存的`uint`。

* 使用`msg.sender`很安全，因為它具有以太坊區塊鏈的安全保障。除非竊取與以太坊地址相關聯的私鑰，否則是没有辦法修改其他人的資料的。

- - -

### Require
`require`會讓函式在執行過程中，如果不滿足某些條件時就會拋出錯誤，並停止執行。範例：

```js
function sayHiToVitalik(string _name) public returns (string) {
  // 比較 _name 是否等於 "Vitalik"。如果不成立，則抛出錯誤並終止程式
  // (Solidity 並不支援原生的字串比較，所以只能透過比較兩個字串的
  // keccak256 雜湊值來進行判斷)
  require(keccak256(_name) == keccak256("Vitalik"));
  // 如果返回 true，則繼續執行下面程式碼
  return "Hi!";
}
```
* 如果這樣調用函式：`sayHiToVitalik("Vitalik")`，會回傳`Hi！`。但如果調用時使用了其他参數，就會抛出錯誤並停止執行。

* 因此，在調用一個函式之前，用`require`驗證前置條件是非常有必要的。

- - -

### Inheritance 繼承
當程式碼過於冗長時，最好將程式碼和邏輯拆分到多個不同的合約中，以便於管理。`inheritance`(繼承)可以讓程式碼容易管理維護：

```js
contract Doge {
  function catchphrase() public returns (string) {
    return "So Wow CryptoDoge";
  }
}

contract BabyDoge is Doge {
  function anotherCatchphrase() public returns (string) {
    return "Such Moon BabyDoge";
  }
}
```

* `BabyDoge`繼承了`Doge`，這代表當你編譯和部署`BabyDoge`後，它將可以訪問`catchphrase()`和`anotherCatchphrase()`和其他在`Doge`中定義的其他公共函式。

- - -

### Import

在 Solidity 中，當你有多個文件並且想把一個文件導入另一個文件時，可以使用`import`：

```js
import "./someothercontract.sol";
contract newContract is SomeOtherContract {

}
```

- - -

### Storage vs Memory (Data location)

在 Solidity 中，有兩個地方可以儲存變數 —— `storage` 或 `memory`。

* `Storage` 變數是指永久儲存在區塊鏈中的變數。 
* `Memory` 變數則是臨時的，當外部函式對某合約調用完成時，暫存型變數即被移除。 
你可以把它想像成，資料儲存在你電腦的硬碟或是RAM中的關係。

大多數時候你都用不到這些關鍵字，默認情況下 Solidity 會自動處理它們。 State變數（在函式之外宣告的變數）默認為“**storage**”形式，並永久寫入區塊鏈；而在函式内部宣告的變數是“**memory**”形式，在函式調用結束後就會消失。

然而也有一些情况下，需要手動宣告資料型態，主要用於處理函式内的`structs`和`arrays`時：

```js
contract SandwichFactory {
  struct Sandwich {
    string name;
    string status;
  }

  Sandwich[] sandwiches;

  function eatSandwich(uint _index) public {
    // Sandwich mySandwich = sandwiches[_index];

    // ^ 看上去很直接，但 Solidity 會拋出警告訊息
    // 這裡要明確定義 `storage` 或者 `memory`

    // 定義為 `storage`:
    Sandwich storage mySandwich = sandwiches[_index];
    // `mySandwich` 是指向 `sandwiches[_index]`的指針
    mySandwich.status = "Eaten!";
    // 這將永久把 `sandwiches[_index]` 變為區塊鏈上的storage

    // 定義為`memory`:
    Sandwich memory anotherSandwich = sandwiches[_index + 1];
    // 這樣 `anotherSandwich` 就僅僅只是memory裡的副本
    anotherSandwich.status = "Eaten!";
    // 這將僅僅修改臨時變數，對 `sandwiches[_index + 1]` 没有任何影響
    // 不過你可以這樣做:
    sandwiches[_index + 1] = anotherSandwich;
    // 如果你想把副本的改動保存回區塊鏈storage
  }
}
```

- - -

### Internal and External

除`public`和`private`屬性之外，Solidity 還使用了另外兩個描述函式可見性的修飾詞：`internal`（内部）和`external`（外部）。

* `internal`和`private`類似，不過，如果某個合約繼承自其父合約，這個合約就可以訪問父合約中定義的“**`internal`**”函式。

* `external`與`public`類似，只不過這些函式只能在合約之外調用 - 它們不能被合約内的其他函式調用。

宣告函式`internal`或`external`類型的語法，與聲明`private`和`public`類型相同：

```js
contract Sandwich {
  uint private sandwichesEaten = 0;

  function eat() internal {
    sandwichesEaten++;
  }
}

contract BLT is Sandwich {
  uint private baconSandwichesEaten = 0;

  function eatWithBacon() public returns (string) {
    baconSandwichesEaten++;
    // 因為eat() 是internal 的，所以我們能在這裡調用
    eat();
  }
}
```

- - -

### Interface
如果我們的合約需要和區塊鏈上的其他的合約互動，則需先定義一個`interface`(接口)。

先舉一個簡單的例子。 假設在區塊鏈上有這麼一個合約：

```js
contract LuckyNumber {
  mapping(address => uint) numbers;

  function setNum(uint _num) public {
    numbers[msg.sender] = _num;
  }

  function getNum(address _myAddress) public view returns (uint) {
    return numbers[_myAddress];
  }
}
```

這是個很簡單的合約，可以用它來儲存自己的幸運號碼。 這樣其他人就可以通過你的地址找到你的幸運號碼。

現在假設我們有一個外部合約，使用`getNum`函式可讀取其中的資料。

首先，我們定義`LuckyNumber`合約的`interface`(接口)：

定義`interface`(接口)：

```js
contract NumberInterface {
  function getNum(address _myAddress) public view returns (uint);
}
```

在合約中這樣用：

```js
contract MyContract {
  address NumberInterfaceAddress = 0xab38...;
  // ^ 這是FavoriteNumber合約在以太坊上的地址
  NumberInterface numberContract = NumberInterface(NumberInterfaceAddress);
  // 现在變數 `numberContract` 指向另一個合約對象

  function someFunction() public {
    // 现在我們可以調用在那個合約中聲明的 `getNum`函式:
    uint num = numberContract.getNum(msg.sender);
  }
}
```

* 通過這種方式，只要將合約的可見性設置為`public`(公共)或`external`(外部)，它們就可以與以太坊區塊鏈上的任何其他合約進行互動。

- - -

### Handling Multiple Return Values

`getKitty`是我們所看到的第一個回傳多個值的函式。我們來看看是如何處理的：

```js
function multipleReturns() internal returns(uint a, uint b, uint c) {
  return (1, 2, 3);
}

function processMultipleReturns() external {
  uint a;
  uint b;
  uint c;
  // 這樣做批次賦值:
  (a, b, c) = multipleReturns();
}

// 或者如果只想回傳其中一個變數:
function getLastReturnValue() external {
  uint c;
  // 可以對其他字段留空:
  (,,c) = multipleReturns();
}
```

- - -

### If statements

if語句的語法在 Solidity 中，與在 JavaScript 中差不多：

```js
function eatBLT(string sandwich) public {
  // 看清楚了，在比較字串時，需要比較他們的 keccak256 雜湊值
  if (keccak256(sandwich) == keccak256("BLT")) {
    eat();
  }
}
```
