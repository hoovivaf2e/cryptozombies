# Beginner to Intermediate Smart Contracts

## Lesson 4 - Zombie Battle System
- `zombiefactory.sol` - 建立殭屍工廠，並生產殭屍軍隊。每隻殭屍的DNA都是隨機生成的，並且擁有自己獨特的外觀。
- `zombiefeeding.sol` - 通過獵食其他生物(crypto kitty)，擴張殭屍軍團。
- `zombiehelper.sol` - 編寫 DApp 時必知的：智能合約的所有權、Gas的花費、程式碼優化、和程式碼安全性。
- `ownable.sol` - 所有權方法合約。
- `zombieattack.sol` - 創建一個殭屍作戰系統、也學習 payable 函式、學習如何開發可以接收其他玩家付款的DApp。

## Learn

### Payable

* 一個函式可以定義多個修飾符

```js
function test() external view onlyOwner anotherModifier { /* ... */ }
```

**payable**
payable 方法可以讓使用者在調用函式的同時付錢給另一個合約。

來看個例子：
```js
contract OnlineStore {
  function buySomething() external payable {
    // 檢查以確定發送0.001以太:
    require(msg.value == 0.001 ether);
    // 如果為真，一些用來向函式調用者發送數字内容的邏輯:
    transferThing(msg.sender);
  }
}
```
* `msg.value`是一種可以查看向合約發送了多少以太的方法。
* `ether`是內建的單位。

有些人會從 web3.js 調用這個函式 (從DApp的前端)，像這樣 :

```js
// 假設 `OnlineStore` 在以太坊上指向你的合約:
OnlineStore.buySomething({from: web3.eth.defaultAccount, value: web3.utils.toWei(0.001)})
```
* `value`用來指定發送多少以太

- - -

### Withdraws

在你發送以太之後，它將被儲存進以合約的以太坊帳戶中，並凍結在那裡，除非你添加一個函式從合約中把以太提出。

你可以寫一個函式從合約中取出以太，類似這樣：

```js
contract GetPaid is Ownable {
  function withdraw() external onlyOwner {
    owner.transfer(this.balance);
  }
}
```
你可以通過`transfer`函式向一個地址發送以太， 然後`this.balance`將返回當前合約約儲存了多少以太。

你可以通過`transfer`向任何以太坊地址付錢。 比如，你可以有一個函式在`msg.sender`超額付款的時候退錢給他們：

```js
uint itemFee = 0.001 ether;
msg.sender.transfer(msg.value - itemFee);
```
或者在一個有賣家和買家的合約中，你可以把賣家的地址儲存起來，當有人買了它的東西時，把買家支付的錢發送给它`seller.transfer(msg.value)`。

- - -

### Random Numbers

**用keccak256產生隨機數**
Solidity 中最好的隨機數產生器是`keccak256`雜湊函數.

我們可以這樣產生一些隨機數：

```js
// 產生一個0到100的隨機數:
uint randNonce = 0;
uint random = uint(keccak256(now, msg.sender, randNonce)) % 100;
randNonce++;
uint random2 = uint(keccak256(now, msg.sender, randNonce)) % 100;
```

* 這個方法首先拿到`now`的時間戳記、`msg.sender`、以及一個自增數`nonce` (一個僅會被使用一次的數)。

* 然後利用`keccak`把輸入的值轉變為一個雜湊值，再將雜湊值轉換為`uint`，然後利用`% 100`取最後兩位，就產生了一個0到100之間的隨機數了。

**這個方法很容易被不誠實的節點攻擊**
在以太坊上, 當你在和一個合約上調用函式的時候, 你會把它廣播给一個節點或者在網路上的 transaction 節點們。 網路上的節點將收集很多事務, 試著成為第一個解決计算密集型數學問題的人，作為“工作證明”，然後將“工作證明”(Proof of Work, PoW)和事務一起作為一個 block 發布在網路上。

一旦一個節點解決了一個PoW, 其他節點就會停止嘗試解決這個 PoW, 並驗證其他節點的事務列表是有效的，然後接受這個節點轉而嘗試解決下一個節點。

這就讓我們的隨機數函式變得可利用了。

假設我們有一個硬幣翻轉合約 —— 正面你赢雙倍錢，反面你輸掉所有的錢。假如它使用上面的方法來決定是正面還是反面 (random >= 50 算正面, random < 50 算反面)。

如果我正執行一個節點，我可以只對我自己的節點發布一個事務，且不分享它。我可以執行硬币翻轉方法來偷窺我的輸贏 — 如果我輸了，我就不把這個事務包含進我要解決的下一個區塊中。我可以一直執行這個方法，直到我赢得了硬幣翻轉並解決了下一個區塊，然後獲利。

**該如何在以太坊上安全地產生隨機數**
* 一個方法是利用`oracle`來訪問以太坊區塊鏈之外的隨機數函式。

- - -

### Refactoring Common Logic
* 將需要多次調用的檢查邏輯獨立成一個`modifier`，來清理程式碼並避免重複造輪子。

