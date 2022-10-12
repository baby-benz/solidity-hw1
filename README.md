# Задача 1

https://github.com/gnosis/MultiSigWallet/blob/master/contracts/MultiSigWallet.sol - сделать, чтобы с баланса
multisig-контракта за одну транзакцию не могло бы уйти больше, чем 66 ETH

## git diff

```diff
@@ -232,6 +232,8 @@ contract MultiSigWallet {
-            txn.executed = true;
-            if (external_call(txn.destination, txn.value, txn.data.length, txn.data))
-                Execution(transactionId);
-            else {
-                ExecutionFailure(transactionId);
-                txn.executed = false;
+            if (txn.value <= 66 ether) {
+                txn.executed = true;
+                if (external_call(txn.destination, txn.value, txn.data.length, txn.data))
+                    Execution(transactionId);
+                else {
+                    ExecutionFailure(transactionId);
+                    txn.executed = false;
+                }
```

# Задача 2

https://github.com/OpenZeppelin/openzeppelin-contracts/blob/f2112be4d8e2b8798f789b948f2a7625b2350fe7/contracts/token/ERC20/ERC20.sol
\- сделать, чтобы токен не мог бы быть transferred по субботам

## git diff

```diff
@@ -41,2 +41,2 @@ contract ERC20 is Context, IERC20 {
+    uint256 private constant SECONDS_PER_DAY = 24 * 60 * 60;
+
@@ -214,6 +216,11 @@ contract ERC20 is Context, IERC20 {
require(_balances[sender] >= amount, "ERC20: transfer amount exceeds balance");
-        _balances[sender] -= amount;
-        _balances[recipient] += amount;

-        emit Transfer(sender, recipient, amount);
+        if (_dayOfWeek(block.timestamp) == 6) {
+            _balances[sender] -= amount;
+            _balances[recipient] += amount;
+
+            emit Transfer(sender, recipient, amount);
+        } else {
+            revert("Tokens are not available on Saturdays");
+        }
  }
  @@ -298,2 +305,10 @@ contract ERC20 is Context, IERC20 {
  function _beforeTokenTransfer(address from, address to, uint256 amount) internal virtual { }
+
+    /**
+     * @dev Returns 1 for Monday and 7 for Sunday
+     */
+    function _dayOfWeek(uint256 timestamp) internal pure returns (uint256 dayOfWeek) {
+        uint _days = timestamp / SECONDS_PER_DAY;
+        dayOfWeek = (_days + 3) % 7 + 1;
+    }
```

# Задача 3

https://github.com/mixbytes/solidity/blob/076551041c420b355ebab40c24442ccc7be7a14a/contracts/token/DividendToken.sol -
сделать чтобы платеж в ETH принимался только специальной функцией, принимающей помимо ETH еще комментарий к платежу (
bytes[32]). Простая отправка ETH в контракт запрещена

## git diff

```diff
@@ -21 +21 @@ contract DividendToken is StandardToken, Ownable {
-    event Deposit(address indexed sender, uint256 value);
+    event Deposit(address indexed sender, uint256 value, bytes32 comment);
@@ -41,4 +41 @@ contract DividendToken is StandardToken, Ownable {
-        if (msg.value > 0) {
-            emit Deposit(msg.sender, msg.value);
-            m_totalDividends = m_totalDividends.add(msg.value);
-        }
+        revert();
@@ -58,0 +56,6 @@ contract DividendToken is StandardToken, Ownable {
+    function deposit(bytes32 comment) public payable {
+        require(msg.value > 0);
+        emit Deposit(msg.sender, msg.value, comment);
+        m_totalDividends = m_totalDividends.add(msg.value);
+    }
+
```