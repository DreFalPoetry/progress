# progress
<br/>

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

<br/>

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
<br/>

### ABL函数 ABL Functions
---------------------
1. Date functions 日期函数

    | 函数 | 参数 | 返回值 | 
    |--| --| --|
    | DAY| DATE | INTEGER — The day of the month |
    | MONTH | DATE | INTEGER  — The month of the year | 
    | YEAR | DATE | INTEGER — The year | 
    | WEEKDAY| DATE | INTEGER — The day of the week, startingwith 1 for Sunday |
    | TIME | none | INTEGER — The number of seconds since midnight|
    | TODAY | none | DATE — Today’s date |

2.  List functions 列表函数

    | 函数 | 参数 | 返回值 | 
      |--| --| --|
      | ENTRY| element <br>  list <br> delimiter | CHARACTER |
      | LOOKUP|  element <br>  list <br> delimiter | INTEGER |

      ```
      def var str as char init "11,232,323".
      entry: 
        disp entry(2, str).
      结果返回："232"

      LOOKUP: 
        disp lookup("232", str).
      结果返回：2
      ```

3. ABL string manipulation functions 字符处理函数

    | 函数 | 参数 | 返回值 | 
    |--| --| --|
    | FILL | expression AS CHARACTER <br> repeat-count AS INTEGER | CHARACTER
    | INDEX | source AS CHARACTER,<br>target AS CHARACTER,<br>starting-point AS INTEGER | INTEGER |
    | LEFT-TRIM | string AS CHARACTER<br>trim-chars AS CHARACTER | CHARACTER |
    | LENGTH | string AS CHARACTER,<br>type AS CHARACTER | INTEGER |
    | R-INDEX | source AS CHARACTER,<br>target AS CHARACTER,<br>starting-point AS INTEGER | INTEGER
    | REPLACE | source AS CHARACTER,<br>from-string ASCHARACTER,<br>to-string AS CHARACTER | CHARACTER |
    | RIGHT-TRIM| string AS CHARACTER,<br>trim-chars AS CHARACTER | CHARACTER |
    | SUBSTRING | source AS CHARACTER,<br>position AS INTEGER,<br>length AS INTEGER,<br>type AS CHARACTER | CHARACTER |
    | TRIM | string AS CHARACTER,<br>trim-chars AS CHARACTER | CHARACTER |


    ```
    def var str as char init "2".
    def var str_p as char init "11223".

    fill: disp fill(str, 3).
    结果返回："222"

    index: disp index(str_p, str).
    结果返回：3
    index: disp index(str_p, str, 4).
    结果返回：4

    LEFT-TRI: def var str as char init " 2 1".
    返回结果："2 1"

    LENGTH:  disp length(str).
    返回结果：1

    r-index: disp r-index(str_p, str).
    结果返回：4 //从后往前

    REPLACE: disp r-index(str_p, str, 'n').
    结果返回："11nn3"

    RIGHT-TRIM: 
    def var str as char format "x(4)" init "2 d ".
    def var str_p as char init "11223".
    disp right-trim(str) + "11".
    结果返回："2 d11"

    SUBSTRING: disp substring("11223", 2, 3).
    结果返回："122"

    TRIM: disp  "1" + trim(" 2 3 ") +  "4".
    结果返回："12 34"

    ```
<br/>

### 程序运算
---------------------

1. 运算操作 （ +，-，*，/）

    例子：
    如果客户的信用额度小于未偿余额的两倍，则显示信用限制与未偿余额的比率。否则，显示该客户的订单。
    ```
    DEFINE VARIABLE cMonthList AS CHARACTER NO-UNDO
    INITIAL "JAN,FEB,MAR,APR,MAY,JUN,JUL,AUG,SEP,OCT,NOV,DEC".

    FOR EACH Customer NO-LOCK WHERE Customer.State = "NH" BY Customer.City:
      DISPLAY Customer.CustNum Customer.Name Customer.City.
      IF Customer.CreditLimit < 2 * Customer.Balance THEN
        DISPLAY "Credit Ratio:" Customer.CreditLimit / Customer.Balance.
      ELSE
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

2. 算数内置函数

    | 函数 | 参数 | 返回值 | 
    |--| --| --|
    | ABSOLUTE | value AS INTEGER or DECIMAL | INTEGER or DECIMAL| 
    | EXP 指数函数 | | |
    | LOG 对数函数 | | |
    | MAXIMUM | | |
    | MINIMUM | | |
    | MODULO 余数 | | |
    | RANDOM | | |
    | ROUND | | |
    |  SQRT  平方根 | | |
    |  TRUNCATE | | |


    ```
    ABSOLUTE:
    def var num as deci init -1.23.
    disp absolute(num).
    结果： 1.23

    EXP:
    def var num as deci init 4.
    disp exp(num, 3).
    结果： 64

    ROUND： 
    def var a as deci init 1.236. 
    disp ROUND(a, 2).
    结果： 1.24

    SQRT： 
    def var a as deci init 3.16. 
    disp SQRT(a).
    结果: 1.78

    TRUNCATE: 
    def var a as deci init 3.16. 
    disp TRUNCATE(a,1).
    结果: 3.10

    ```

3. 转存为 `.r` 文件

* 打开New Procedure Window. 

  运行：`COMPILE h-CustSample.p SAVE.` 在文件目录中生成 `.r` 文件

* `COMPILE source-procedure SAVE INTO directory.`

    例如
    `compile c:\abc\test.p save into c:\000\`
    在c盘000文件夹中生成了一个`test.r`文件

    在编辑器 执行 `run c:\000\test.r` 可以得到运行结果

<br/>

### 运行ABL程序 Running ABL Procedures
---------------------

1. 运行子程序
    
    语法
    ```
    RUN procedure-name [ (parameters) ].
    ```
    示例：

    `run E:/p/test.p.`

<br/>

### 使用外部和内部程序 Using external and internal procedures 
---------------------

1. 编写内部程序

    * 以 `PROCEDURE`开头跟程序名称
    * 定义过程中使用的参数，并定义参数类型 输入 输出 或者 输入-输出
    `INPUT`, `OUTPUT`, `INPUT-OUTPUT`
    * 参数名以p开头以便识别
    * 以`END PROCEDURE`结束 提高代码可读性

    <br/>

    示例：定义一个名为 ` calcDays` 的过程
    ```
    PROCEDURE calcDays:
      DEFINE INPUT PARAMETER pdaShip AS DATE NO-UNDO.
      DEFINE OUTPUT PARAMETER piDays AS INTEGER NO-UNDO.
      piDays = TODAY - pdaShip.
    END PROCEDURE.
    ```
2. 运行内部程序 并展示运行结果
    ```
    RUN calcDays (INPUT Order.ShipDate, OUTPUT iDays).
    DISPLAY iDays LABEL "Days" FORMAT "ZZZ9".
    ```
3. 涉及语法：为变量赋值

    Syntax：简介语法
    ```
    result-value = IF logical-expression
        THEN value-if-true ELSE value-if-false
    ```
    示例：
    ```
    piDays = IF pdaShip = ? THEN 0 ELSE TODAY - pdaShip.
    ```
4. 返回语句和返回值 
  在过程结尾放置返回语句
    ```
    RETURN [ return-value ].
    ```
    如果在调用过程中没有返回值，则返回空字符串

    返回语句在过程中是可选的
5. 向过程中添加注释
    以 `/*` 开头以 `*/` 结束
6. 完整示例

    `h-CustSample.p`
    ```
    /* h-CustSample.p — shows a few things about ABL */
    DEFINE VARIABLE cMonthList AS CHARACTER NO-UNDO
      INITIAL "JAN,FEB,MAR,APR,MAY,JUN,JUL,AUG,SEP,OCT,NOV,DEC".
    DEFINE VARIABLE iDays AS INTEGER NO-UNDO.
    /* First display each Customer from New Hampshire: */
    FOR EACH Customer NO-LOCK WHERE Customer.State = "NH" BY Customer.City:
      DISPLAY Customer.CustNum Customer.Name Customer.City.
      /* Show the Orders unless the Credit Limit is less than twice the balance. */
      IF Customer.CreditLimit < (2 * Customer.Balance) THEN
        DISPLAY "Credit Ratio:" Customer.CreditLimit / Customer.Balance.
      ELSE
      FOR EACH Order OF Customer NO-LOCK:
        DISPLAY
          Order.OrderNum LABEL "Order"   
          Order.OrderDate
          Order.ShipDate FORMAT "99/99/99" WITH CENTERED. 
          /* Show the month as a three-letter abbreviation, 
          along with the number of days since the order was shipped. */
        IF Order.ShipDate NE ? THEN
          DISPLAY ENTRY(MONTH(Order.ShipDate), cMonthList) LABEL "Month".
        RUN calcDays (INPUT Order.ShipDate, OUTPUT iDays).
        DISPLAY iDays LABEL "Days" FORMAT "ZZZ9".
      END.
    END.

    PROCEDURE calcDays:
      /* This calculates the number of days since the Order was shipped.*/ 
      DEFINE INPUT PARAMETER pdaShip AS DATE NO-UNDO.
      DEFINE OUTPUT PARAMETER piDays AS INTEGER NO-UNDO.
      piDays = IF pdaShip = ? THEN 0 ELSE TODAY - pdaShip.
    END PROCEDURE.
    ```   


    
































