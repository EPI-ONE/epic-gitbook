# Transaction

The structure of a transaction in EPI is very similar to Bitcoin's transaction. The difference lies in the ScriptSig in which EPI has simpler script. Transactions in EPI also has Status which indicates whether a transaction is valid, invalid or an unknown status.

## Structure

![Transaction Structure](../.gitbook/assets/screenshot-2019-03-28-at-1.06.21-pm.png)

## Status

* Conditions for an invalid transaction:
  * Missing input or output for redemption of reward
  * Redeem value greater than reward
  * Double redemption
  * Number of sigOps is greater than `MAX_BLOCK_SIGOPS`
  * Signature failed
  * Transaction fees to large \(greater than `MAX_MONEY`\)
* Invalid transaction will catch exception and terminate
* Only valid transaction will be allowed to accumulate reward

This is an enum that describes the underlying reason the transaction was created. It's useful for rendering wallet GUIs more appropriately.

```java
public enum Status {
        UNKNOWN(0),
        VALID(1),
        INVALID(2);

        private int value;
        private static Map<Integer, Status> map = new HashMap<>();

        private Status(int value) {
            this.value = value;
        }

        static {
            for (Status Status : Status.values()) {
                map.put(Status.value, Status);
            }
        }

        public static Status valueOf(int Status) {
            return map.get(Status);
        }

        public int getValue() {
            return value;
        }
    }
```

