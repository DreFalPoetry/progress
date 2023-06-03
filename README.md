# progress

### 语法
---------------------
### basic 
1. `FOR EACH`
  ``` 
  FOR EACH Customer:
  ```
2. `DISPLAY`
```
FOR EACH Customer:
DISPLAY Customer.
```
3. `WHERE`
```
FOR EACH Customer WHERE Customer.State = "NH":
DISPLAY Cust-Num Name City.
```
4. `BY`
```
FOR EACH Customer WHERE Customer.State = "MA" BY Customer.City:
```

### nested blocks
---------------------
1. `OF`  -- 表关联
```
FOR EACH Customer NO-LOCK WHERE Customer.State = “NH” BY  Customer.City:
  DISPLAY Customer.CustNum Customer.Name Customer.City.
  FOR EACH Order OF Customer NO-LOCK:
    DISPLAY Order.OrderNum Order.OrderDate Order.ShipDate.
  END.
END.
```
2. `WITH CENTERED` -- 设置frame显示

```
FOR EACH Customer NO-LOCK WHERE Customer.State = “NH” BY  Customer.City:
  DISPLAY Customer.CustNum Customer.Name Customer.City.
  FOR EACH Order OF Customer NO-LOCK:
    DISPLAY Order.OrderNum Order.OrderDate Order.ShipDate  ` WITH CENTERED.
  END.
END.
```
3. LABEL/FORMAT   -- 修改标签和格式
```
FOR EACH Customer NO-LOCK WHERE Customer.State = "NH" BY Customer.City:
  DISPLAY Customer.CustNum Customer.Name Customer.City.
  FOR EACH Order OF Customer NO-LOCK:
    DISPLAY
      Order.OrderNum LABEL "Order" 
      Order.OrderDate
      Order.ShipDate FORMAT "99/99/99" WITH CENTERED.
  END.
END.
```
4. 操作符介绍 comparison operators

    |  关键字  |  标识  |  解释  |
    | ----- | ----| ----|
    | EQ | = | 相等|
    |NE | <> | 不等于 |
    | GT | > | 大于 |
    | LT | < | 小于 |
    |GE | >= | 大于等于 | 
    |LE | <=  | 小于等于 | 
    | BEGINS | | 以此字符串开头|
    | MATCHES | | 相匹配的字符串（"*"表示全部 "."表示没有匹配到的）
    | CONTAINS ？？ | | 包含 | 


### 变量和数据类型
--------------
#### 语法： `DEFINE VARIABLE varname AS datatype.`

1. 基础的数据类型

    |  名称  |  默认展示格式  |  默认值  |
    | ----| -- |-- |
    | CHARACTER | X(8) |  ""(空字符串) | 
    | DATE | 99/99/99 |  ? (Unknown 未知值) | 
    | DECIMAL | ->>,>>9.99 (整数位6位，小数位2位) | 0 |
    | HANDLE ??? | >>>>>>9 | ? | 
    | INTEGER | ->,>>>,>>9 (8位整数) | 0 |
    |LOGICAL| yes/no | no

2. 通用格式符号

    |  格式字符  |  含义  |
    | ----| -- |
    | X | 单个字符 |
    | N | 单个数字或字母 | 
    | A | 一个字母 | 
    | ！|在输出中转化为大写字母 |
    | 9 | 数字 | 
    | (n) ??? | 数字 -重复前一个格式字符的次数 | 
    | > | 数字达不到设置位置前置位置不显示 |
    | Z | 数字达不到设置位置前置位置为空格 |
    | * | 数字达不到设置位置前置位置为星号 |
    | , | 数值大于1000时的逗号展示 |
    | . | 小数点 | 
    | + ？？？ | 表示整数或负数的符号 | 
    | - ？？？| 表示负数的符号 | 

3. 其他变量限定符

    | 关键字 |  用途 |
    | --- | --- |
    | INITIAL | 变量初始值 |
    |  DECIMALS ？？？ | 小数位数 | 
    | FORMAT | 展示形式 | 
    | LABEL | 变量展示标签 | 
    | COLUMN-LABEL | ??? |
    | EXTENT| |

#### 语法： `DEFINE VARIABLE varname LIKE fieldname.`

继承了其他变量或者字段的格式 标签 初始值 和所有其他属性

#### 示例
1. 示例1
    ```
    DEFINE VARIABLE cMonthList AS CHARACTER NO-UNDO
      INITIAL "JAN,FEB,MAR,APR,MAY,JUN,JUL,AUG,SEP,OCT,NOV,DEC".
      FOR EACH Customer NO-LOCK WHERE Customer.State = "NH" BY Customer.City:
        DISPLAY Customer.CustNum Customer.Name Customer.City.
        FOR EACH Order OF Customer NO-LOCK:
          IF Order.ShipDate NE ? THEN
          DISPLAY 
          Order.OrderNum LABEL "Order" 
          Order.OrderDate
          Order.ShipDate FORMAT "99/99/99" WITH CENTERED.
        END.
      END.
    ```
2. 示例2
```
DEFINE VARIABLE cMonthList AS CHARACTER NO-UNDO
INITIAL "JAN,FEB,MAR,APR,MAY,JUN,JUL,AUG,SEP,OCT,NOV,DEC".
  FOR EACH Customer NO-LOCK WHERE Customer.State = "NH" BY Customer.City:
    DISPLAY Customer.CustNum Customer.Name Customer.City.
    FOR EACH Order OF Customer NO-LOCK:
      DISPLAY 
      Order.OrderNum LABEL "Order" 
      Order.OrderDate
      Order.ShipDate FORMAT "99/99/99" WITH CENTERED.
      IF Order.ShipDate NE ? THEN
      DISPLAY ENTRY(MONTH(Order.ShipDate), cMonthList) LABEL "Month".
    END.
  END.
```







