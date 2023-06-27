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
<br/>

### 程序块和数据访问
---------
1. 块和块属性 Blocks and block properties

    * Every procedure itself constitutes a block, even just the simplest RUN statement executed from an editor window.

      每个过程本身都构成一个块，甚至是从编辑器窗口执行的最简单的RUN语句。
    * Every call to another procedure, whether internal or external, starts another block. An external procedure can contain one or more internal procedures, each of which forms its own block.  

      对另一个过程的调用，无论是内部还是外部，都会启动另一个块。外部过程可以包含一个或多个内部过程，每个内部过程都形成自己的块。
    * ABL statements such as FOR EACH and DO define the start of a new block.

        诸如FOR EACH和DO等ABL语句定义了一个新块的开始。
    * A trigger is a block of its own. 

        触发器是它自己的一个块。

2. 程序块作用域 Procedure block scoping 

    如果在内部过程中定义了变量或其他对象，则它的作用域仅限于该内部过程，在包含它的外部过程中无法使用。可以在内部过程和包含它的外部过程定义相同的变量名。将得到第二个变量，名称相同但是存储位置不同，因此值也不相同。

      示例1（外部程序定义变量在内部过程中也可用）
      ```
      /* mainproc.p */
      /* This is scoped to the whole external procedure. */
      DEFINE VARIABLE cVar AS CHARACTER NO-UNDO INITIAL "Mainproc". 
      RUN subproc.
      PROCEDURE subproc:
        DISPLAY cVar.
      END PROCEDURE.
      ```

      示例2 (在内部过程中定义自己的变量)
      ```
      /* mainproc.p */
      /* This is scoped to the whole external procedure. */
      DEFINE VARIABLE cVar AS CHARACTER NO-UNDO INITIAL "Mainproc".
      RUN subproc.
      DISPLAY cVar.
      PROCEDURE subproc:
        DEFINE VARIABLE cVar AS CHARACTER NO-UNDO.
        cVar = "Subproc".
      END PROCEDURE.
      ```
      You assign a different value to the variable in the subprocedure, but because the subprocedure has its own copy of the variable, that value exists only within the subprocedure. Back in the main procedure, the value Mainproc is not overwritten even after the subprocedure call.

      您可以为子过程中的变量分配不同的值，但由于子过程具有自己的变量副本，因此该值只存在于子过程中。回到主过程中，即使在子过程调用之后，主程序的值也不会被覆盖。
      
      示例3 
      ```
      /* mainproc.p */
      /* This is scoped to the whole external procedure. */
      DEFINE VARIABLE cVar AS CHARACTER NO-UNDO INITIAL "Mainproc". 

      RUN subproc.
      DISPLAY cVar cSubVar.

      PROCEDURE subproc:
        DEFINE VARIABLE cVar AS CHARACTER NO-UNDO.
        DEFINE VARIABLE cSubVar AS CHARACTER NO-UNDO.
        ASSIGN
          cVar = "Subproc"
          cSubVar = "LocalVal".
      END PROCEDURE.
      ```
      Here cSubVar is defined only in the internal procedure, so when you try to display it from the main procedure block you get an error

      这里的cSubVar仅在内部过程中定义，因此当您试图从主过程块中显示它时，您会得到一个错误
    
3. 定义块的语言语句 Language statements that define blocks
    * DO blocks

      Looping with a DO block

      语法
      ```
      DO variable = expression1 TO expression2 [ BY constant ]:
      ```
      示例：将1-5的整数相加 并显示总数
      ```
      DEFINE VARIABLE iCount AS INTEGER NO-UNDO.
      DEFINE VARIABLE iTotal AS INTEGER NO-UNDO.

      DO iCount = 1 TO 5:
        iTotal = iTotal + iCount.
      END.
      DISPLAY iTotal.
      ```

      起始值和结束值可以是表达式，而不仅仅是常量

      示例：
      ```
      DEFINE VARIABLE iCount AS INTEGER NO-UNDO.
      DEFINE VARIABLE iTotal AS INTEGER NO-UNDO INITIAL 1.

      DO iCount = iTotal TO 5:
        iTotal = iTotal + iCount.
      END.
      DISPLAY iTotal.
      ```

    * 使用do语句使用逻辑条件
      ```
      DO WHILE expression:
      ```
      示例：
      ```
      DEFINE VARIABLE iTotal AS INTEGER NO-UNDO INITIAL 1.

      DO WHILE iTotal < 50:
        iTotal = iTotal * 2.
      END.
      DISPLAY iTotal.
      ```

      其余关于do的示例表示有关frame部分暂时无法实验 `p86`


    * FOR blocks： for块    
      * Loops automatically through all the records that satisfy the record set definition in the block 

        自动循环浏览块中满足记录集定义的所有记录

      * Reads the next record from the result set for you as it iterates

        在迭代结果中读取下一个记录

      * Scopes those records to the block

         将这些记录的范围限定到该块

      * Scopes a frame to the block, and you can use the WITH FRAME phrase to specify that frame

        将框架扩展到块，您可以使用 `WITH FRAME` 短语来指定该框架

      * Provides database update services within a transaction

        在事务中提供数据库更新服务

      Sorting records by using the BY phrase（通过使用BY短语对记录进行排序）

      使用by时默认升序。如需降序则添加关键字  DESCENDING
      ```
      BY field [ DESCENDING ] ...
      ```

      Joining tables using multiple FOR phrases (使用多个FOR短语连接表)
      ```
      FOR EACH Customer NO-LOCK WHERE Customer.State = "NH", 
        EACH Order OF Customer NO-LOCK WHERE Order.ShipDate NE ? :
        DISPLAY Customer.Custnum Customer.Name Order.OrderNum Order.ShipDate.
      END.
      ```
      
      不使用限定符：(检索订单和他们的客户 要确保关联表只有唯一匹配)
      ```
      FOR EACH Order NO-LOCK WHERE Order.ShipDate NE ?, Customer OF Order NO-LOCK:
        DISPLAY Order.OrderNum Order.ShipDate Customer.Name.
      END.
      ```

      查看每个客户的第一个订单：
      ```
      FOR EACH Customer NO-LOCK WHERE Customer.State = "NH", 
        FIRST Order OF Customer NO-LOCK:
        DISPLAY Customer.CustNum Customer.Name Order.OrderNum Order.OrderDate.
      END.
      ```
      使用first需要注意第一个订单的排序方式问题:
      所以，就关系到这个问题：Using indexes to relate and sort data （使用索引来关联和排序数据）
      该例子默认按照 ordernum进行升序排序 所以返回的首个订单是orderNum最小的订单
      * Using the USE-INDEX phrase to force a retrieval order：使用使用-索引短语来强制执行检索顺序
        示例：
        ```
        FOR EACH Customer NO-LOCK WHERE Customer.State = "NH", 
          FIRST Order OF Customer NO-LOCK USE-INDEX OrderDate:
          DISPLAY Customer.CustNum Customer.Name Order.OrderNum Order.OrderDate.
        END.
        ```
      * Using the LEAVE statement to leave a block：使用leave语句离开一个块
        ```
        FOR EACH Customer NO-LOCK WHERE Customer.State = "NH" WITH FRAME f:
          DISPLAY Customer.CustNum Customer.Name.
          FOR EACH Order OF Customer NO-LOCK BY Order.OrderDate:
            DISPLAY Order.OrderNum Order.OrderDate WITH FRAME f.
            LEAVE.
          END. /* FOR EACH Order */
        END. /* FOR EACH Customer */
        ```
        在显示客户订单的第一条记录后离开块

      * Using block headers to identify blocks  使用块标头来标识块
        ```
        FOR EACH Customer NO-LOCK WHERE Customer.State = "NH" WITH FRAME f:
          DISPLAY Customer.CustNum Customer.Name.
          OrderBlock:
          FOR EACH Order OF Customer NO-LOCK BY Order.OrderDate:
              DISPLAY Order.OrderNum Order.OrderDate WITH FRAME f.
              LEAVE OrderBlock.
          END. /* FOR EACH Order */
        END. /* FOR EACH Customer */
        ```
        ```
        CustBlock:
        FOR EACH Customer NO-LOCK WHERE Customer.State = "NH" WITH FRAME f:
          DISPLAY Customer.CustNum Customer.Name.
          OrderBlock:
          FOR EACH Order OF Customer NO-LOCK BY Order.OrderDate:
            DISPLAY Order.OrderNum Order.OrderDate WITH FRAME f.
            LEAVE CustBlock.
          END. /* FOR EACH Order */
        END. /* FOR EACH Customer */
        ```
      * Using NEXT, STOP, and QUIT to change block behavior(使用NEXT、STOP和QUIT来更改块行为)

        STOP terminates the current procedure, backs out any active transactions, and returns to the OpenEdge session’s startup procedure or to the Editor. You can intercept a STOP action by including the ON STOP phrase on a block header, which defines an action to take other than the default when the STOP condition occurs.

        STOP终止当前过程，退出任何活动的事务，并返回到OpenEdge会话的启动过程或编辑器。您可以通过在块标头上包含ON STOP短语来拦截STOP操作，该短语定义了在STOP条件发生时要采取的非默认操作。

        QUIT exits from OpenEdge altogether in a run-time environment and returns to the operating system. If you’re running in a development environment, it has a similar effect to STOP and returns to the Editor or to the Desktop. There is also an ON QUIT phrase to intercept the QUIT condition in a block header and define an action to take other than quitting the session.

        QUIT在运行时环境中完全从OpenEdge中退出，并返回到操作系统。如果您是在开发环境中运行的，那么它的效果类似于STOP，并返回到编辑器或桌面。还有一个ON QUIT短语，用于拦截块头中的QUIT条件，并定义除退出会话以外要采取的操作。

    * REPEAT blocks （与for块同样的语法）：`P96`

      使用：重复数据输入操作
      ```
      REPEAT:
        INSERT Customer EXCEPT Customer.Comments WITH 2 COLUMNS.
      END.
      ```

      Using the PRESELECT keyword to get data in advance(使用`PRESELECT`关键字提前获取数据)

    * Data access without looping—the FIND statement(无循环的数据访问-FIND语句)
      Syntax
      ```
      FIND [ FIRST | NEXT| PREV | LAST ] record [ WHERE ...]
            [ USE-INDEX index-name ]
      ```

      示例：
      ```
      FIND FIRST Customer.
      ```
      ```
      FIND FIRST Customer WHERE Customer.State = “NH”.
      ```
      If you include a WHERE clause, the AVM chooses one or more indexes to optimize locating the record. This might have very counter-intuitive results.
      如果包含WHERE子句，AVM将选择一个或多个索引来优化记录的定位。这可能会有非常违反直觉的结果。如下例子展示:
      ```
      FIND FIRST Customer.
      DISPLAY Customer.CustNum Customer.Name Customer.Country.
      ```
      ```
      FIND FIRST Customer WHERE Customer.Country = "USA".
      DISPLAY Customer.CustNum Customer.Name Customer.Country.
      ```

      ```
      FIND FIRST Customer NO-LOCK WHERE Customer.Country = "USA".
      DISPLAY Customer.CustNum Customer.Name Customer.Country.
      REPEAT:
        FIND NEXT Customer NO-LOCK NO-ERROR.
        IF AVAILABLE Customer THEN
          DISPLAY Customer.CustNum Customer.Name FORMAT "x(20)" Customer.Country Customer.PostalCode.
        ELSE LEAVE.
      END.
      ```
      继续索引只在美国的客户：
      ```
      FIND FIRST Customer NO-LOCK WHERE Country = "USA".
      DISPLAY Customer.CustNum Customer.Name Customer.Country.
      REPEAT:
        FIND NEXT Customer NO-LOCK WHERE Customer.Country = "USA" NO-ERROR.
        IF AVAILABLE Customer THEN
          DISPLAY Customer.CustNum Customer.Name FORMAT "x(20)" Customer.Country
      Customer.PostalCode.
        ELSE LEAVE.
      END.
      ```

      Using a USE-INDEX phrase to force index selection
      使用`USE-INDEX`短语来强制进行索引选择
      ```
      FIND FIRST Customer NO-LOCK WHERE Customer.Country = "USA".
      DISPLAY Customer.CustNum Customer.Name Customer.Country.
      REPEAT:
        FIND NEXT Customer WHERE Customer.Country = "USA" USE-INDEX NAME NO-ERROR.
        IF AVAILABLE Customer THEN
          DISPLAY Customer.CustNum Customer.Name FORMAT "x(20)"
      Customer.Country Customer.PostalCode.
        ELSE LEAVE.
      END.
      ```

      Doing a unique FIND to retrieve a single record
      执行唯一的查找操作来检索单个记录
      ```
      FIND Customer WHERE Customer.CustNum = 1025.
      DISPLAY Customer.CustNum Customer.Name Customer.Country.

      速写
      FIND Customer 1025.
      ```

      Using the CAN-FIND function（使用CAN-FIND函数）
      示例：

      如果客户有1997年的订单，则过程将显示客户名称。否则，它将显示文本短语No 1997订单。如果在显示语句中包含该文字值，它将显示在自己的列中，就像它是一个字段或变量一样。要显示它以代替“名称”字段，请使用位置标记符号（@）。
      ```
      FOR EACH Customer NO-LOCK WHERE Customer.Country = "USA":
        IF CAN-FIND(FIRST Order OF Customer WHERE Order.OrderDate < 1/1/98) THEN
          DISPLAY Customer.CustNum Customer.Name.
        ELSE 
          DISPLAY Customer.CustNum "No 1997 Orders" @ Customer.Name.
      END.
      ```

      If you need the Order record itself then you must use a form that returns it to you:
      如果您需要订单记录本身，那么您必须使用一个表单，返回它给您：
      ```
      FOR EACH Customer NO-LOCK WHERE Customer.Country = "USA":
        FIND FIRST Order OF Customer NO-LOCK WHERE OrderDate < 1/1/98 NO-ERROR.
        IF AVAILABLE Order THEN
          DISPLAY Customer.CustNum Customer.Name Order.OrderDate.
        ELSE 
          DISPLAY "No 1997 Orders" @ Customer.Name.
      END.
      ```

      和where语句一同使用 can-find
      ```
      FOR EACH Customer NO-LOCK WHERE Customer.Country = "USA" AND 
        CAN-FIND(FIRST Order OF Customer WHERE Order.OrderDate < 1/1/98):
        DISPLAY Customer.CustNum Customer.Name.
      END.
      ```
<br/>
      
### 记录缓冲区和记录范围
----------

#### 记录缓冲区
-----
1. 语法

    ```
    DEFINE BUFFER <buffer-name> FOR <table-name>.
    ```
    There are many places in complex business logic where you need to have two or more different records from the same table available to your code at the same time, for comparison purposes. This is when you might use multiple different buffers with their own names. Here’s one fairly simple example. In the following procedure, which could be used as part of a cleanup effort for the Customer table, you need to see if there are any pairs of Customers in the same city in the US with zip codes that do not match.

    在复杂的业务逻辑中，有许多地方，您需要同时将同一表中的两个或多个不同的记录可用于代码，以便进行比较。这时，您可以使用多个不同的具有自己名称的缓冲区。这里有一个相当简单的例子。在以下过程中，可以作为客户表清理工作的一部分，您需要查看在美国同一城市是否有邮政编码不匹配的客户对。

    示例：解释查看 `p109`
    ```
    DEFINE BUFFER Customer FOR Customer.
    DEFINE BUFFER OtherCust FOR Customer.

    FOR EACH Customer NO-LOCK WHERE Customer.Country = "USA":
      FIND FIRST OtherCust NO-LOCK
        WHERE Customer.State = OtherCust.State
          AND Customer.City = OtherCust.City
          AND SUBSTRING(Customer.PostalCode, 1,3) NE 
              SUBSTRING(OtherCust.PostalCode, 1,3)
          AND Customer.CustNum < OtherCust.CustNum NO-ERROR.
      IF AVAILABLE OtherCust THEN
        DISPLAY 
          Customer.CustNum 
          Customer.City FORMAT "x(12)" 
          Customer.State FORMAT "xx"
          Customer.PostalCode 
          OtherCust.CustNum
          OtherCust.PostalCode.
    END.
    ```

#### 记录范围 `p111`
-----

### Using Queries 









    
































