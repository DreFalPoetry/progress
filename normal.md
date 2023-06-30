# 常用语句

1. `absolute` FUNCTION

    绝对值
    ```
    /* 变量定义 */
    DEFINE VARIABLE mark-start  AS DECIMAL NO-UNDO.
    DEFINE VARIABLE mark-finish AS DECIMAL NO-UNDO.
    DEFINE VARIABLE units       AS LOGICAL NO-UNDO FORMAT "miles/kilometers". 

    /* FORM ? --一种可以输入 */
    /* WITH FRAME question SIDE-LABELS (question frame 展示 左侧标签)   frame标题为  SKIP(1)空白行数*/
    FORM  
        mark-start LABEL "Mile marker for highway on-ramp" SKIP  
        mark-finish LABEL "Mile marker next to your exit" SKIP(1) 
        units LABEL "Measure in <m>iles or <k>ilometers" SKIP(1)  
        WITH FRAME question SIDE-LABELS   
        /*  frame标题为  */
        TITLE "This program calculates distance driven.". 

    /* 设置 question frame中的这三个值 */    
    UPDATE mark-start mark-finish units WITH FRAME question. 

    /* 将结果展示到 answer frame中*/
    DISPLAY
        "You have driven" ABSOLUTE(mark-start - mark-finish) units 
        WITH NO-LABELS FRAME answer.
        
    ```


2. `accum` FUNCTION

    Returns the value of an aggregate expression that is calculated by an ACCUMULATE or aggregate phrase of a DISPLAY statement

    返回由DISPLAY语句的ACCUMULATE或aggregate短语计算的聚合表达式的值
    示例：
    ```
    FOR EACH Order NO-LOCK:  
    DISPLAY Order.OrderNum Order.CustNum Order.OrderDate Order.PromiseDate    
            Order.ShipDate.  
      FOR EACH OrderLine OF Order:    
        DISPLAY OrderLine.LineNum OrderLine.ItemNum OrderLine.Qty      
                OrderLine.Price (OrderLine.Qty * OrderLine.Price) LABEL "Ext Price".    
        ACCUMULATE OrderLine.Qty * OrderLine.Price (TOTAL).    
        DISPLAY (ACCUM TOTAL OrderLine.Qty * OrderLine.Price) LABEL "Accum Total".  
      END.  
    DISPLAY (ACCUM TOTAL OrderLine.Qty * OrderLine.Price) LABEL "Total".
    END.
    DISPLAY (ACCUM TOTAL OrderLine.Qty * OrderLine.Price) LABEL "Grand Total"  WITH ROW 1.
    ```

3. `ACCUMULATE` statement

    Calculates one or more aggregate values of an expression during the iterations of a block. Use the ACCUM function to access the result of this accumulation.

    在块迭代期间计算表达式的一个或多个聚合值。使用ACCUM函数访问此累积的结果。

    示例1：显示了所有Customer的CreditLimit的平均值 总数量 最大值
    ```
    FOR EACH Customer NO-LOCK:  
      ACCUMULATE Customer.CreditLimit (AVERAGE COUNT MAXIMUM).
    END. 
    DISPLAY "MAX-CREDIT STATISTICS FOR ALL CUSTOMERS:" SKIP(2)        
            "AVERAGE =" (ACCUM AVERAGE Customer.CreditLimit) SKIP(1)        
            "MAXIMUM =" (ACCUM MAXIMUM Customer.CreditLimit) SKIP(1)        
            "NUMBER OF CUSTOMERS =" (ACCUM COUNT Customer.CreditLimit) SKIP(1)        
            WITH NO-LABELS.
    ```

    示例2： 计算 每条记录的OnHand*Price 所占总数的百分比
    ```
    FOR EACH Item NO-LOCK:  
      ACCUMULATE Item.OnHand * Item.Price (TOTAL).
    END. 

    FOR EACH Item NO-LOCK BY Item.OnHand * Item.Price DESCENDING:  
        DISPLAY Item.ItemNum Item.OnHand Item.Price Item.OnHand * Item.Price     
            LABEL "Value" 100 * (Item.OnHand * Item.Price) /     
            (ACCUM TOTAL Item.OnHand * Item.Price) LABEL "Value %".
    END.
    ```

    示例3：根据销售代表 通过客户城市排序 汇总 销售代表城市下的销售额 他的总销售总额
    ```
    FOR EACH Customer NO-LOCK BREAK BY Customer.SalesRep BY Customer.Country:
    ACCUMULATE Customer.Balance 
        (TOTAL BY Customer.SalesRep BY Customer.Country).
    DISPLAY Customer.SalesRep WHEN FIRST-OF(Customer.SalesRep) 
        Customer.Country Customer.Name Customer.Balance.
    IF LAST-OF(Customer.Country) THEN
        DISPLAY ACCUM TOTAL BY Customer.Country Customer.Balance
            COLUMN-LABEL "Country!Total".
    IF LAST-OF(Customer.SalesRep) THEN DO:
        DISPLAY Customer.SalesRep ACCUM TOTAL BY Customer.SalesRep
            Customer.Balance COLUMN-LABEL "SalesRep!Total".
        DOWN 1.
    END.
    END.
    ```

4. `ADD-INTERVAL` function

    Adds a time interval to, or subtracts a time interval from, a DATE, DATETIME, or DATETIME-TZ value, and returns the new value.

    向DATE、DATETIME或DATETIME- tz值添加时间间隔或从中减去时间间隔，并返回新值。
    ```
    ADD-INTERVAL (datetime, interval-amount, interval-unit)
    ```
    示例：
    ```
    disp ADD-INTERVAL (today, 1, "days").
    ```

5. `ASC` function

    Converts a character expression representing a single character into the corresponding ASCII (or internal code page) value, returned as an INTEGER.

    将表示单个字符的字符表达式转换为相应的ASCII(或内部代码页)值，并作为INTEGER返回。

    计算以每个字母A-Z开头的客户名的数量和其他不以 a-z开头的客户数量记为other
    ```
    DEFINE VARIABLE ix   AS INTEGER NO-UNDO.
    DEFINE VARIABLE jx   AS INTEGER NO-UNDO.
    DEFINE VARIABLE ltrl AS INTEGER NO-UNDO EXTENT 27. 

    FOR EACH Customer NO-LOCK:  
        ix = ASC(SUBSTRING(Customer.Name,1,1)).  
        IF ix < ASC("A") or ix > ASC("Z") THEN ix = EXTENT(ltrl).  
        ELSE ix = ix - ASC("A") + 1.  
        ltrl[ix] = ltrl[ix] + 1.
    END.
    
    DO jx = 1 TO EXTENT(ltrl) WITH NO-LABELS USE-TEXT:  
        IF jx <= 26 THEN    
            DISPLAY CHR(ASC("A") + jx - 1) @ ltr-name AS CHARACTER FORMAT "x(5)".  
        ELSE     
            DISPLAY "Other" @ ltr-name.  
        DISPLAY ltrl[jx].
    END.
    ```

6. `ASSIGN` statement

    示例1：
    ```
    REPEAT:
      PROMPT-FOR Customer.CustNum.
      FIND Customer USING Customer.CustNum NO-ERROR.
      IF NOT AVAILABLE Customer THEN DO:
        CREATE Customer.
        ASSIGN Customer.CustNum.
      END. 
      UPDATE Customer WITH 2 COLUMNS.
    END.
    ```
    示例2：输入订单号和订单行号 找到这个订单行
    设置 两个变量
    找到 订单号和输入的新号匹配的订单 设置订单行的订单号为新订单号 行号为新行号
    ```
    DEFINE VARIABLE neword LIKE order-line.order-num LABEL "New Order".
    DEFINE VARIABLE newordli LIKE order-line.line-num LABEL "New Order Line".
    REPEAT:
      PROMPT-FOR OrderLine.OrderNum OrderLine.LineNum.
      FIND OrderLine USING OrderLine.OrderNum AND OrderLine.LineNum.
      SET neword newordli.
      FIND Order WHERE Order.OrderNum = neword.
      ASSIGN 
        OrderLine.OrderNum = neword
        OrderLine.LineNum = newordli.
    END.
    ```
7. `AVAILABLE` function

    Returns a TRUE value if the record buffer you name contains a record and returns a FALSE value if the record buffer is empty.

    如果您指定的记录缓冲区包含一条记录，则返回TRUE值;如果记录缓冲区为空，则返回FALSE值。
    ```
    REPEAT:  
      PROMPT-FOR Item.ItemNum.  
      FIND Item USING ItemNum NO-ERROR.  
      IF AVAILABLE Item THEN     
          DISPLAY Item.ItemName Item.Price.  
      ELSE     
          MESSAGE "Not found".
    END.
    ```
8. for each *** break by
    break 根据某个字段进行分组 需要在break后使用by关键字做排序
    ```
    FOR EACH Customer BREAK BY Customer.State:  
    DISPLAY Customer.State Customer.Name     
            Customer.CreditLimit (TOTAL BY state).
    END.
    ```

9. `BUFFER-COPY` statement
    记录缓冲区复制
    ```
   BUFFER-COPY source [ { EXCEPT | USING } field ... ]  
    TO target [ ASSIGN assign-expression ... ] [ NO-LOBS ] [ NO-ERROR ]
    ```
10. `by` Option
    
    Performs aggregation for break groups if you use the BREAK option in a FOR EACH block header. 

    如果在for EACH块标头中使用break选项，则对break组执行聚合。
    
    ```
    FOR EACH Customer NO-LOCK BREAK BY Customer.Country:  
    DISPLAY Customer.Name Customer.Country Customer.Balance   
      (SUB-TOTAL BY Customer.country).
    END.
    ```
    作为排序依据
    ```
    FOR EACH Customer BY Customer.CreditLimit BY Customer.Name
    ```

11. `CAN-DO` function

12. `CAN-FIND` function

    Returns a TRUE value if a record is found that meets the specified FIND criteria;  otherwise it returns FALSE.  
    如果找到符合指定FIND条件的记录，则返回TRUE值;否则返回FALSE。
    
13. `caps`

    Converts any lowercase characters in a CHARACTER or LONGCHAR expression to uppercase characters, and returns the result.

    将CHARACTER或LONGCHAR表达式中的任何小写字符转换为大写字符，并返回结果。

    ```
    REPEAT:  
      PROMPT-FOR Customer.CustNum.  
      FIND Customer USING Customer.CustNum.  
      UPDATE Customer.Name Customer.Address Customer.City Customer.State.  
      Customer.State = CAPS(Customer.State).  
      DISPLAY Customer.State.
    END.
    ```

14. `CASE` statement

    基于单个表达式的值提供多分支决策。
    ```
    DEFINE VARIABLE pay-stat AS INTEGER NO-UNDO INITIAL 1.
    UPDATE pay-stat VIEW-AS RADIO-SET
      RADIO-ITEM unpaid 1 LABEL "Unpaid"
      RADIO-ITEM part 2 LABEL "Partially paid"
      RADIO-ITEM paid 3 LABEL "Paid in full".

    CASE pay-stat:
      WHEN 1 THEN
      MESSAGE "This account is unpaid.".
      WHEN 2 THEN 
      MESSAGE "This account is partially paid.".
      WHEN 3 THEN 
      MESSAGE "This account is paid in full.".
    END CASE.
    ```

15. `CHR` function
    ```
    DEFINE VARIABLE ix     AS INTEGER   NO-UNDO.
    DEFINE VARIABLE letter AS CHARACTER NO-UNDO FORMAT "X(1)" EXTENT 26. 

    DO ix = 1 TO 26:  
        letter[ix] = CHR((ASC("A")) - 1 + ix).
    END. 

    DISPLAY SKIP(1) letter WITH 2 COLUMNS NO-LABELS  
        TITLE "T H E  A L P H A B E T".
    ```


16. `CLEAR` statement

    清除帧中所有填充字段的数据。它还清除框架中所有小部件的颜色，启用的填充物除外。

    ```
    DEFINE VARIABLE a AS CHARACTER NO-UNDO INITIAL "xxxxxxxx".
    DEFINE VARIABLE b AS DATE      NO-UNDO INITIAL TODAY.
    DEFINE VARIABLE c AS DECIMAL   NO-UNDO INITIAL "-12,345.67".
    DEFINE VARIABLE d AS INTEGER   NO-UNDO INITIAL "-1,234,567".
    DEFINE VARIABLE e AS LOGICAL   NO-UNDO INITIAL TRUE.

    DISPLAY "This illustrates the default formats for the different data types"
        SKIP (2) WITH CENTERED ROW 4 NO-BOX FRAME head.
    FORM "CHARACTER default format is ""x(8)"" " a SKIP
        "DATE default format is 99/99/99         " b SKIP
        "DECIMAL default format is ->>,>>9.99    " c SKIP
        "INTEGER default format is ->,>>>,>>9    " d SKIP
        "LOGICAL default format is yes/no        " e TO 55 SKIP
          WITH ROW 12 NO-BOX NO-LABELS CENTERED FRAME ex.
          
          
    REPEAT:  
        DISPLAY a b c d WITH FRAME ex.  
        MESSAGE "Do you want to put in some values?"  
        UPDATE e.  
        IF e THEN DO:    
            CLEAR FRAME ex NO-PAUSE.    
            SET a b c d WITH FRAME ex.  
        END.  
        ELSE LEAVE.
    END.
    ```

17. `COLUMN-LABEL` option

    Names the label you want to display above the variable data in a frame that uses column labels.  If you want the label to use more than one line (a stacked label), use an exclamation point (!) in the label to indicate where to break the line. 

    在使用列标签的框架中，指定要在变量数据上方显示的标签。如果希望标签使用多行(堆叠标签)，请在标签中使用感叹号(!)来指示换行的位置。

    ```
    DEFINE VARIABLE credit-percent AS INTEGER NO-UNDO  
    COLUMN-LABEL "Enter   !percentage!increase ".

    FOR EACH Customer:
    DISPLAY Customer.Name Customer.CreditLimit.
    SET credit-percent.
    Customer.CreditLimit = (Customer.CreditLimit *     
        (credit-percent / 100)) + Customer.CreditLimit.
    DISPLAY Customer.CreditLimit @ new-credit LIKE Customer.CreditLimit    
        LABEL "New max cred".
    END.
    ```

    ```
    FOR EACH Customer NO-LOCK:  
    DISPLAY Customer.Name COLUMN-LABEL "Customer!Name"    
    Customer.SalesRep COLUMN-LABEL "Name of!Sales!Representative".
    END.
    ```

18. `COLUMN` Option

    ```
    DEFINE VARIABLE paid-owed AS DECIMAL NO-UNDO.
    DEFINE VARIABLE bal-label AS CHARACTER NO-UNDO FORMAT "x(20)".
    FOR EACH Customer NO-LOCK:
    paid-owed = Customer.Balance.
    IF paid-owed < 0 /* Customer has a credit */ THEN DO:
    paid-owed = - paid-owed.
    bal-label = "Customer Credit".
    END.
    ELSE bal-label = "Unpaid balance".
    DISPLAY Customer.CustNum Customer.Name paid-owed LABEL " " WITH 1 DOWN.
    IF Customer.Balance < 0 THEN
    PUT SCREEN COLOR MESSAGES ROW 2 COLUMN 36 bal-label.
    ELSE 
    PUT SCREEN ROW 2 COLUMN 36  bal-label.
    END.
    ```

19. `COMPILE` statement  

20. `CONNECT` statement

21. `CONNECTED` function

    Tells whether a database is connected.  If logical name is the logical name or alias is the alias of a connected database, the CONNECTED function returns TRUE;  otherwise, it returns FALSE.

    表示数据库是否已连接。如果逻辑名称是逻辑名称或别名是连接数据库的别名，则connected函数返回TRUE;否则，返回FALSE。

    ```
    IF CONNECTED("sports2000") THEN RUN r-dispcu.p.
    ```

22. `CREATE` statement
    
    Creates a record in a table, sets all the fields in the record to their default initial values, and moves a copy of the record to the record buffer.

    在表中创建一条记录，将记录中的所有字段设置为默认初始值，并将记录的副本移动到记录缓冲区。

    ```
    REPEAT:
      CREATE Order.
      UPDATE Order.OrderNum Order.CustNum 
        VALIDATE(CAN-FIND(Customer OF Order), "Customer does not exist")
        Order.CustNum Order.OrderDate.
      REPEAT:
        CREATE OrderLine.
        OrderLine.OrderNum = Order.OrderNnum.
        UPDATE OrderLine.LineNum OrderLine.ItemNum 
          VALIDATE(CAN-FIND(Item OF OrderLine), "Item does not exist")
          OrderLine.Qty OrderLine.Price.
      END.
    END.
    ```




```
DEFINE QUERY q-order FOR Customer, Order, OrderLine, Item. 
OPEN QUERY q-order FOR EACH Customer,  
    EACH Order OF Customer,  
    EACH OrderLine OF Order,  
    EACH Item OF OrderLine NO-LOCK.
    
GET FIRST q-order. 

DO WHILE AVAILABLE Customer:  
    DISPLAY Customer.CustNum Customer.Name SKIP    
        Customer.Phone SKIP    
        Order.OrderNum Order.OrderDate SKIP    
        OrderLine.LineNum OrderLine.Price OrderLine.Qty SKIP    
        Item.ItemNum Item.ItemName SKIP    
        Item.CatDesc VIEW-AS EDITOR SIZE 50 BY 2 SCROLLBAR-VERTICAL    
        WITH FRAME ord-info CENTERED SIDE-LABELS TITLE "Order Information".   
        
    /* Allow scrolling, but not modification, of CatDesc. */  
    ASSIGN Item.CatDesc:READ-ONLY IN FRAME ord-info = TRUE         
            Item.CatDesc:SENSITIVE IN FRAME ord-info = TRUE.  
    PAUSE.  
    GET NEXT q-order.
END. /* DO WHILE AVAILABLE Customer */ 
```


