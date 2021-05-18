# Beginner to Intermediate Smart Contracts

## Lesson 6 - App Front-ends & Web3.js
- `zombiefactory.sol` - 建立殭屍工廠，並生產殭屍軍隊。每隻殭屍的DNA都是隨機生成的，並且擁有自己獨特的外觀。
- `zombiefeeding.sol` - 通過獵食其他生物(crypto kitty)，擴張殭屍軍團。
- `zombiehelper.sol` - 編寫 DApp 時必知的：智能合約的所有權、Gas的花費、程式碼優化、和程式碼安全性。
- `ownable.sol` - 所有權方法合約。
- `zombieattack.sol` - 創建一個殭屍作戰系統、也學習 payable 函式、學習如何開發可以接收其他玩家付款的DApp。
- `zombieownership.sol` - 查看殭屍擁有者、交易殭屍等，改寫 ERC721 標準。
- `erc721.sol` - ERC721 標準。
- `safemath.sol` - 安全運算。
- `index.html` - 使用 Web3.js 來為 DApp 創建一個基本的前端介面，和zombie合約互動。

## Learn

### Web3.js

Web3.js是以太坊基金會發布的 JavaScript library。

以太坊網路是由節點組成的，每一個節點都包含了一份區塊鏈的副本。當你想要調用某個智能合約中的一個方法時，就必須從其中一個節點中查詢並告訴節點:

1. 那個智能合約的地址
2. 你想要調用的方法
3. 需要傳入的參數

以太坊節點只認識一種叫做`JSON-RPC`的語言。這種語言直接讀起來並不好懂。當你想調用某個合約的方法時，需要發送的查詢語句如下：
```js
{"jsonrpc":"2.0","method":"eth_sendTransaction","params":[{"from":"0xb60e8dd61c5d32be8058bb8eb970870f07233155","to":"0xd46e8dd67c5d32be8058bb8eb970870f07244567","gas":"0x76c0","gasPrice":"0x9184e72a000","value":"0x9184e72a","data":"0xd46e8dd67c5d32be8d46e8dd67c5d32be8058bb8eb970870f072445675058bb8eb970870f072445675"}],"id":1}
```

Web3.js提供了方便易懂的javascript介面，使用者只要調用Web3.js就可以與以太坊節點互動。

```js
// 用 NPM
npm install web3

// 用 Yarn
yarn add web3

// 用 Bower
bower install web3
```

- - -

### Web3 Provider
以太坊是許多節點構成，各節點擁有同一份資料。在`Web3.js`裡設置`Web3 Provider`就是在說明我們要和哪個節點交互來處理讀跟寫。就好比在 Web 應用中為你的 API 設置`Web server`的 url 一樣。

你可以運行你自己的以太坊節點作為 Provider，也可以使用第三方的服務 — Infura。Infura可以讓你更輕鬆，它可以讓你不必為了要提供使用者一個應用而去維護一個以太坊節點。

**Infura**

Infura 是一個服務，它維護了很多以太坊節點並提供了一個緩存層來實現快速讀取。你可以用他們的 API 免費訪問Infura。用 Infura 作為節點提供者，你可以不用自己運行節點就能很可靠地向以太坊發送、接收訊息。

`Infura`作為`Web3 Provider`可以這樣用：
```js
var web3 = new Web3(new Web3.providers.WebsocketProvider("wss://mainnet.infura.io/ws"));
```

- - -

### Talking to Contracts
Web3.js 需要兩個東西才能和合約對話: 合約的**地址**和**ABI**。

**Contract ABI**
ABI 意為應用二進位接口（Application Binary Interface）。 基本上，它是以 JSON 格式表示合約的方法，告訴 Web3.js 如何以合同理解的方式格式化函數調用。

**實例化 Web3.js Contract**
```js
var myContract = new web3js.eth.Contract(myABI, myContractAddress);
```
- - 

### 調用和合約函數 call & send
Web3.js 有两個方法來調用我們合約的函數: call and send.

**Call**
* call 用來調用`view`和`pure`函數。它只運行在本地節點，不會在區塊鏈上創建事務。

* `view`和`pure`函數是唯讀的，並不會改變區塊鏈的狀態，也不會消耗任何gas。使用者也不會被要求用MetaMask對事務簽名。

* 使用 Web3.js 可以這樣做： 呼叫 `myMethod` 方法，並傳入參數123：

```js
myContract.methods.myMethod(123).call()
```

**Send**
* `send`將創建一個事務並改變區塊鏈上的資料。你需要用 `send` 來調用任何非 `view` 或者 `pure` 的函數。

* `send` 將要求使用者支付gas，並會彈出對話框請求使用者使用 Metamask 對事務簽名。在我們使用 Metamask 作為 web3 provider 的同時，所有這一切都會在調用 send() 的時候自動發生。

* 使用 Web3.js 可以這樣做：

```js
myContract.methods.myMethod(123).send({ from: userAccount, value: web3js.utils.toWei("0.001","ether") })
```

- - 

### 調用 Payable 函數

一個`wei`是以太的最小單位 — `1 ether` 等于 `10^18 wei`

Web3.js 轉換 ether / wei：
```js
// 把 1 ETH 轉換成 Wei
web3js.utils.toWei("1", "ether");
```

在我們的 DApp 里，設置了 `levelUpFee = 0.001 ether`，所以調用 levelUp 方法的時候，可以讓使用者用以下的方式同時發送 0.001 以太:

```js
cryptoZombies.methods.levelUp(zombieId).send({ from: userAccount, value: web3js.utils.toWei("0.001","ether") })
```

- - 

### Subscribing to Events

在 Web3.js里， 你可以訂閱一個事件，這樣你的 Web3 提供者可以在每次事件發生后觸發你的一些程式碼邏輯：

```js
cryptoZombies.events.NewZombie()
.on("data", function(event) {
  let zombie = event.returnValues;
  console.log("一個新殭屍誕生了！", zombie.zombieId, zombie.name, zombie.dna);
}).on('error', console.error);
```

**使用 indexed**
為了篩選僅和當前使用者相關的事件，Solidity 合約將必須使用 indexed 關鍵字，就像我們在實現 ERC721 中的 Transfer 事件中那樣：

```js
event Transfer(address indexed _from, address indexed _to, uint256 _tokenId);
```
在這種情況下，因為 `_from` 和 `_to` 都是 `indexed`，這代表我們可以在前端事件監聽中過濾事件

```js
cryptoZombies.events.Transfer({ filter: { _to: userAccount } })
.on("data", function(event) {
  let data = event.returnValues;
  // 當前使用者更新了一個殭屍！更新界面來顯示
}).on('error', console.error);
```

使用 event 和 indexed 對於監聽合約中的更改並將其反應到 DApp 的前端界面中是非常有用的做法。

**查詢過去的事件**
我們甚至可以用 `getPastEvents` 查詢過去的事件，並用過濾器 `fromBlock` 和 `toBlock` 給 Solidity 一個事件日誌的時間範圍("block" 在這里代表以太坊區塊編號）：
```js
cryptoZombies.getPastEvents("NewZombie", { fromBlock: 0, toBlock: 'latest' })
.then(function(events) {
  // events 是可以用來遍歷的 `event` 物件
  // 這段程式碼將回傳從開始以來創建的殭屍列表
});
```
