# Beginner to Intermediate Smart Contracts

## Lesson 5 - ERC721 & Crypto-Collectibles
- `zombiefactory.sol` - 建立殭屍工廠，並生產殭屍軍隊。每隻殭屍的DNA都是隨機生成的，並且擁有自己獨特的外觀。
- `zombiefeeding.sol` - 通過獵食其他生物(crypto kitty)，擴張殭屍軍團。
- `zombiehelper.sol` - 編寫 DApp 時必知的：智能合約的所有權、Gas的花費、程式碼優化、和程式碼安全性。
- `ownable.sol` - 所有權方法合約。
- `zombieattack.sol` - 創建一個殭屍作戰系統、也學習 payable 函式、學習如何開發可以接收其他玩家付款的DApp。
- `zombieownership.sol` - 查看殭屍擁有者、交易殭屍等，改寫 ERC721 標準。
- `erc721.sol` - ERC721 標準。
- `safemath.sol` - 安全運算。

## Learn

### Tokens on Ethereum

`token`【代幣】在以太坊上就是個遵循一些規則的智能合約。
在智能合約内部，通常有一個mapping，用於追蹤每個地址還有多少餘額：
```js
mapping(address => uint256) balances
```

基本上，`token`只是一個追蹤誰擁有多少該代幣的合約，和一些可以讓使用者將他們的代幣轉移到其他地址的函式。

#### ERC20 Token Standard

ERC20 代幣標準共享具有相同名稱的函式，他們可以用相同的方法溝通。
這代表如果你建立的應用能夠與一個 ERC20 代幣溝通，那麼也可以與任何 ERC20 代幣進行交流互動。這樣一來，你就可以輕鬆地將更多的代幣添加或轉移到你的應用中，而不用再另外寫程式。

ERC20標準：

```js
contract ERC20Interface {
    function totalSupply() public constant returns (uint);
    function balanceOf(address tokenOwner) public constant returns (uint balance);
    function allowance(address tokenOwner, address spender) public constant returns (uint remaining);
    function transfer(address to, uint tokens) public returns (bool success);
    function approve(address spender, uint tokens) public returns (bool success);
    function transferFrom(address from, address to, uint tokens) public returns (bool success);

    event Transfer(address indexed from, address indexed to, uint tokens);
    event Approval(address indexed tokenOwner, address indexed spender, uint tokens);
}
```


#### ERC721 Token Standard

**Non-fungible Token**
最大的特點就是他不能互換，因為每個代幣都被認為是唯一且不可分割的、是獨一無二的。你只能以整個單位交易它們，並且每個單位都有唯一的 ID。

ERC721標準：

```js
contract ERC721 {
  event Transfer(address indexed _from, address indexed _to, uint256 indexed _tokenId);
  event Approval(address indexed _owner, address indexed _approved, uint256 indexed _tokenId);

  function balanceOf(address _owner) external view returns (uint256);
  function ownerOf(uint256 _tokenId) external view returns (address);
  function transferFrom(address _from, address _to, uint256 _tokenId) external payable;
  function approve(address _approved, uint256 _tokenId) external payable;
}
```

實現代幣合約：
1. 將interface複製到自己的專案中
2. 在自己的合約中import它，`import "./erc721.sol";`。
3. 讓自己的合約繼承自它，然後重寫每個方法。

- - 

### Overflows and Underflows


什麼是**溢出(overflow)**?

假設我們有一個`uint8`，代表只能儲存`8 bit`的資料。換句話說，能夠儲存的最大數字是255(十進位表示：2^8 - 1 = 255)，或是`11111111`(二進位表示)。

看看下面的範例：

```js
uint8 number = 255;
number++;     // number: 0
```

在這例子中導致了溢出，雖然只加了1，但是 number 出乎意料的等於 0。就好比你給二進位 11111111 加1, 他就會變成 00000000 (11111111 + 1 原本應該是 100000000，但因為`uint8`只有`8 bit`，所以變成 00000000)；也就像是時鐘從 23:59 走向 00:00 這樣。

**下溢(underflow)**也類似，如果你從一個等於 0 的 uint8 減 1，它就會變成 255，因為 uint 是無符號的(也就是說他不能是負數)。

所以必須在合約中添加一些保護機制，防止我們的 DApp 出現什麼異常情況。為了防止這種情況，通常會使用SafeMath語法來做運算。OpenZeppelin 的 SafeMath library 有四個方法 — add， sub， mul，以及 div。


- - -

### SafeMath Part 1

OpenZeppelin SafeMath library：

```js
library SafeMath {

  function mul(uint256 a, uint256 b) internal pure returns (uint256) {
    if (a == 0) {
      return 0;
    }
    uint256 c = a * b;
    assert(c / a == b);
    return c;
  }

  function div(uint256 a, uint256 b) internal pure returns (uint256) {
    // assert(b > 0); // Solidity automatically throws when dividing by 0
    uint256 c = a / b;
    // assert(a == b * c + a % b); // There is no case in which this doesn't hold
    return c;
  }

  function sub(uint256 a, uint256 b) internal pure returns (uint256) {
    assert(b <= a);
    return a - b;
  }

  function add(uint256 a, uint256 b) internal pure returns (uint256) {
    uint256 c = a + b;
    assert(c >= a);
    return c;
  }
}
```
`assert` 和 `require` 很像，若結果為否它就會抛出錯誤。`assert` 和 `require` 的差別在於 `require` 若失敗，會將剩下的 gas 返還給使用者；`assert` 則不會。所以通常比較常用到 `require`，`assert` 只在程式碼可能出現嚴重錯誤的時候使用，比如 uint 溢出。

簡言之， SafeMath 的 add， sub， mul， 和 div 方法只做簡單的四則運算，然後在發生溢出或下溢的時候抛出錯誤。


- - -

### SafeMath Part 2

如果有的參數是uint16、有的參數是uint32，將這些参數傳入 SafeMath 的 add 方法的話，實際上並不會防止溢出，因为他會把這些變數都轉換成 uint256。
型態不同，也會導致問題，所以需要再實現兩個libraty來防止 uint16 和 uint32 溢出或下溢。可以參考`safemath.sol`。

- - -

### Comments
Solidity 裡的註釋和 JavaScript 相同。

```js
// 這是一個單行註釋，可以理解為給自己或者別人看的筆記
```

還有多行註釋

```js
/* 这是一个多行註釋。
最好為你的合約的每個方法添加註釋來解釋它的行為。這樣其他開發者（或你自己）可以很快地理解程式碼而不需要逐行閱讀所有程式碼。
*/
```

Solidity 使用的註釋標準是 `natspec` 格式，看起來像這樣：

```js
/// @title 一個簡單的基處運算合約
/// @author H4XF13LD MORRIS
/// @notice 為合約添加一個乘法
contract Math {
  /// @notice 兩個數相乘
  /// @param x 第一個 uint
  /// @param y  第二個 uint
  /// @return z  (x * y) 的结果
  /// @dev 這個方法不檢查溢出
  function multiply(uint x, uint y) returns (uint z) {
    // 這只是個普通的註釋，不會被 natspec 解釋
    z = x * y;
  }
}
```

* `@title` 標題 和 `@author` 作者。
* `@notice` 是向使用者解釋這個方法或合約是做什麼的。 
* `@dev` 是向開發者解釋更多細節。
* `@param` 和 `@return` 用來描述這個方法需要傳入什麼參數以及回傳什麼值。
* 不用每次都用上所有標籤，所有標籤都是可選的，不過最少寫個 `@dev` 解釋每個方法是做什麼的。