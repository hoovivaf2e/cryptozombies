# Beginner to Intermediate Smart Contracts

## Lesson 3 - Advanced Solidity Concepts
- `zombiefactory.sol` - 建立殭屍工廠，並生產殭屍軍隊。每隻殭屍的DNA都是隨機生成的，並且擁有自己獨特的外觀。
- `zombiefeeding.sol` - 通過獵食其他生物(crypto kitty)，擴張殭屍軍團。
- `zombiehelper.sol` - 編寫 DApp 時必知的：智能合約的所有權、Gas的花費、程式碼優化、和程式碼安全性。
- `ownable.sol` - 所有權方法合約。

## Learn

### Ownable Contracts

下面是一個 Ownable 合約的例子： 來自`OpenZeppelin`，Solidity 庫的`Ownable`合約。`OpenZeppelin`是主打安保和社區審查的智能合約庫，你可以在自己的DApps中引用。

範例：

```js
/**
 * @title Ownable
 * @dev The Ownable contract has an owner address, and provides basic authorization control
 * functions, this simplifies the implementation of "user permissions".
 */
contract Ownable {
  address public owner;
  event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);

  /**
   * @dev The Ownable constructor sets the original `owner` of the contract to the sender
   * account.
   */
  constructor() {
    owner = msg.sender;
  }

  /**
   * @dev Throws if called by any account other than the owner.
   */
  modifier onlyOwner() {
    require(msg.sender == owner);
    _;
  }

  /**
   * @dev Allows the current owner to transfer control of the contract to a newOwner.
   * @param newOwner The address to transfer ownership to.
   */
  function transferOwnership(address newOwner) public onlyOwner {
    require(newOwner != address(0));
    emit OwnershipTransferred(owner, newOwner);
    owner = newOwner;
  }
}
```

- `constructor` 構造函式：`constructor`不是必須的，`constructor`只會在合約最初被建立的時候執行一次。

Ownable 合約基本都會：

1. 合約建立後，構造函式先執行，將`owner`設置為`msg.sender`（其部署者）。

2. 為它加上一個修飾符`onlyOwner`，它會限制陌生人的訪問，將訪問某些函式的權限鎖定在`owner`上。

3. 允許將合約所有權轉讓給他人。

- - -

### onlyOwner - Function Modifier

**Modifier**

函式修飾符看起來跟函式没什麼不同，關鍵字`modifier`明確告知編譯器modifier(修飾符)，而不是個function(函式)。它不能像函式那樣直接被調用，只能被添加到函式定義的末尾，以改變函式的行為。

看看範例中的`onlyOwner`:
```js
/**
 * @dev 調用者不是'主人'，就會抛出異常
 */
modifier onlyOwner() {
  require(msg.sender == owner);
  _;
}
```

onlyOwner 函式修飾符用法：

```js
contract MyContract is Ownable {
  event LaughManiacally(string laughter);

  //注意！ `onlyOwner` :
  function likeABoss() external onlyOwner {
    LaughManiacally("Muahahahaha");
  }
}
```
* 注意`likeABoss`函式上的`onlyOwner`修飾符。 當你調用`likeABoss`時，首先執行`onlyOwner`中的程式碼， 執行到 `onlyOwner`中的`_;`語句時，程式再回傳並執行`likeABoss`中的程式碼。

* 儘管函式修飾符也可以應用到各種場合，但最常見的還是放在函式執行之前添加`require`檢查。

* 因為給函式添加了修飾符 onlyOwner，使得唯有合約的主人（也就是部署者）才能調用它。

- - -

### Gas

* 在 Solidity 中，使用者每次執行你的 DApp 都需要支付一定的 gas，gas 可以用以太幣購買，因此，使用者每次跑 DApp 都得花費以太幣。

* 一個 DApp 收取多少 gas 取決於功能邏輯的複雜程度。每個操作背後，都在計算完成這個操作所需要的計算資源（比如，儲存資料就比做個加法運算貴得多），一次操作所需要花費的 gas 等於這個操作背後的所有運算花銷的總和。

* 由於運行DApp需要花費使用者的真金白銀，所以在以太坊中程式碼的程式語言比其他任何程式語言都更强調優化。同樣的功能，使用笨拙的程式碼開發的程式，比起經過精巧優化的程式碼所需的花費更高，這顯然會給成千上萬的使用者帶來大量不必要的開銷。

**為什麼要用 gas 來驅動？**

* 以太坊就像一個巨大、緩慢、但非常安全的電腦。當你運行一個程式的時候，網路上的每一個節點都在進行相同的運算，以驗證它的輸出 —— 這就是所謂的“去中心化”。由於數以千計的節點同時在驗證著每個功能的運行，這可以確保它的資料不會被監控，或者被刻意修改。

* 可能會有使用者用無限循環堵塞網路，或用密集運算來占用大量的網路資源，為了防止這種事情的發生，以太坊的建立者為以太坊上的資源制定了價格，想要在以太坊上運算或者儲存，你需要先付費。

**省 gas 的招數：結構封装 （Struct packing）**

* 除了基本版的`uint`外，還有其他變種 uint：`uint8`，`uint16`，`uint32`等。

* 通常情況下我們不會考慮使用 uint 變種，因為無論如何定義 uint 的大小，Solidity 為它保留`256 bits`的儲存空間。例如，使用`uint8`而不是`uint(uint256)`不會為你節省任何 gas。

* 除非，把 uint 绑定到 struct 裡面。如果一個 struct 中有多個 uint，則盡可能使用較小的 uint，Solidity 會將這些 uint 打包在一起，從而占用較少的儲存空間。例如：

```js
struct NormalStruct {
  uint a;
  uint b;
  uint c;
}

struct MiniMe {
  uint32 a;
  uint32 b;
  uint c;
}

// 因為使用了結構打包，`mini` 比 `normal` 占用的空間更少
NormalStruct normal = NormalStruct(10, 20, 30);
MiniMe mini = MiniMe(10, 20, 30); 
```

所以，當 uint 定義在一個 struct 中的時候，盡量使用最小的整數子類型以節約空間。 並且把同樣類型的變數放一起（即在 struct 中將把變數按照類型依次放置），這樣 Solidity 可以將儲存空間最小化。例如，有两個 struct：

`uint c; uint32 a; uint32 b;` 和 `uint32 a; uint c; uint32 b;`

前者比後者需要的gas更少，因為前者把uint32放一起了。

- - -

### 時間單位

* Solidity 使用自己的本地時間單位。

* 變數 now 將回傳當前的unix時間戳記（自1970年1月1日以來經過的秒數）。

* Solidity 還包含秒(seconds)，分鐘(minutes)，小時(hours)，天(days)，周(weeks) 和 年(years) 等時間單位。它們都會轉換成對應的秒數放入 uint 中。

下面是一些使用時間單位的實用案例：

```js
uint lastUpdated;

// 將'上次更新時間' 設置為 '現在'
function updateTimestamp() public {
  lastUpdated = now;
}

// 如果到上次`updateTimestamp` 超過5分鐘，回傳 'true'
// 不到5分鐘回傳 'false'
function fiveMinutesHavePassed() public view returns (bool) {
  return (now >= (lastUpdated + 5 minutes));
}
```

- - -

### 將結構體(structs)作為参數傳入
由於結構體(`structs`)的儲存指針可以以参數的方式傳遞給一個 private 或 internal 的函式，因此結構體(`structs`)可以在多個函式之間相互傳遞。

遵循這樣的語法：
```js
function _doStuff(Zombie storage _zombie) internal {
  // do stuff with _zombie
}
```

- - -

### Modifier with arguments

函式修飾符也可以帶参數。例如：
```js
// 儲存使用者年齡的mapping
mapping (uint => uint) public age;

// 限定使用者年齡的修飾符
modifier olderThan(uint _age, uint _userId) {
  require(age[_userId] >= _age);
  _;
}

// 可以用如下参數調用`olderThan` 修飾符:
function driveCar(uint _userId) public olderThan(16, _userId) {
  // ...
}
```
`olderThan`修飾符可以像函式一樣接收参數，而`driveCar`把参數傳遞給它的修飾符的。



- - -

### “view” 函式不花 “gas”

* 當使用者從外部調用一個view函式，是不需要支付一分 gas 的。這是因為 view 函式不會真正改變區塊鏈上的任何資料 - 它們只是讀取。執行view函式只需要查詢你的本地以太坊節點，不需要在區塊鏈上建立一個事務（事務需要運行在每個節點上，因此會花費gas）。

* 但是如果一個 view 函式在另一個函式的内部被調用，而調用函式與 view 函式的不屬於同一個合約，也會產生調用成本。這是因為如果主調函式在以太坊建立了一個事務，它仍然需要逐個節點去驗證。所以標 view 函式只有在外部調用時才是免費的。

- - -

### storage非常昂貴
* Solidity 使用storage(儲存)是相當昂貴的，”寫入“操作尤其貴。

* 這是因為，無論是寫入還是更改一段資料，都將永久性地寫入區塊鏈。”永久性“啊！需要在全球數千個節點的硬碟上存入這些資料，随著區塊鏈的增長，拷貝份數更多，儲存量也就越大。這是需要成本的！

* 為了降低成本，不到萬不得已，避免將資料寫入儲存。這也包含沒效率的程式邏輯 - 比如每次調用一個函式，都需要在 memory(内存) 中重建一個陣列，而不是簡單地將上次計算的陣列給儲存下來以便快速找尋。

* 在大多數程式語言中，遍歷大資料集合都是昂貴的。但是在 Solidity 中，使用一個標記了`external view`的函式，遍歷比 storage 要便宜太多，因為 view 函式不會產生任何花銷。

**在memory中聲明陣列**

在陣列後面加上 memory關鍵字，表明這個陣列是僅僅在memory中建立，不需要寫入外部儲存，並且在函式調用結束時它就解散了。與在程式結束時把資料保存進 storage 的做法相比，memory運算可以大大節省gas開銷。

以下是申明一個内存陣列的例子：

```js
function getArray() external pure returns(uint[]) {
  // 初始化一個長度為3的memory陣列
  uint[] memory values = new uint[](3);
  // 賦值
  values.push(1);
  values.push(2);
  values.push(3);
  // 回傳陣列
  return values;
}
```

- - -

### For 循環

函式中使用的陣列是運行時在memory中通過 for 循環時建構的，而不是預先建立在storage中的。

為什麼要這樣做呢？

為了實現 getZombiesByOwner 函式，一種“無腦式”的解決方案是在 ZombieFactory 中存入”主人“和”殭屍軍團“的mapping。

```js
mapping (address => uint[]) public ownerToZombies
```

然後我們每次建立新殭屍時，執行`ownerToZombies[owner].push（zombieId）`將其添加到主人的殭屍陣列中。而`getZombiesByOwner`函式也非常简單：

```js
function getZombiesByOwner(address _owner) external view returns (uint[]) {
  return ownerToZombies[_owner];
}
```

**這個做法有問题**
做法倒是簡單。可是如果我們需要一個函式來把一頭殭屍轉移到另一個主人名下，又會發生什麼是呢？

這個“換主”函式要做到：

1. 將殭屍push到新主人的`ownerToZombies`陣列中
2. 從舊主的`ownerToZombies`陣列中移除殭屍
3. 將舊主殭屍陣列中“換主殭屍”之後的的每頭殭屍都往前挪一位，把挪走“換主殭屍”後留下的“空位”填上
4. 將陣列長度減 1

但是第三步實在是太貴了！因為每挪動一頭殭屍，我們都要執行一次寫入操作。如果一個主人有20頭殭屍，而第一頭被挪走了，那為了保持陣列的順序，我們得做19個寫入操作。

由於寫入儲存是 Solidity 中最費 gas 的操作之一，使得換主函式的每次調用都非常昂貴。更糟糕的是，每次調用的時候花費的 gas 都不同！具體還取決於使用者在原主軍團中的殭屍頭數，以及移走的殭屍所在的位置。以至於使用者都不知道應该支付多少 gas。

由於從外部調用一個 view 函式是免費的，我們也可以在`getZombiesByOwner`函式中用一個for循環遍歷整個殭屍陣列，把屬於某個主人的殭屍挑出來建構殭屍陣列。那麼`transfer`函式將會便宜得多，因為我們不需要挪動儲存裡的殭屍陣列重新排序，總體上這個方法會更便宜，雖然有點反直覺。

**使用 for 循環**
for循環的語法在 Solidity 和 JavaScript 中類似。

來看一個建立偶數陣列的例子：

```js
function getEvens() pure external returns(uint[]) {
  uint[] memory evens = new uint[](5);
  // 在新陣列中記錄序列号
  uint counter = 0;
  // 在循環從1迭代到10：
  for (uint i = 1; i <= 10; i++) {
    // 如果 `i` 是偶數...
    if (i % 2 == 0) {
      // 把它加入偶數陣列
      evens[counter] = i;
      //索引加一， 指向下一個空的‘even’
      counter++;
    }
  }
  return evens;
}
```

這個函式將回傳一個形為 [2,4,6,8,10] 的陣列。
