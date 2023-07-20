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

23. `DATE` function
    ```
    DEFINE VARIABLE cnum AS CHARACTER NO-UNDO FORMAT "x(3)".
    DEFINE VARIABLE cdate AS CHARACTER NO-UNDO FORMAT "x(16)".
    DEFINE VARIABLE iday AS INTEGER NO-UNDO.
    DEFINE VARIABLE imon AS INTEGER NO-UNDO.
    DEFINE VARIABLE iyr AS INTEGER NO-UNDO.
    DEFINE VARIABLE ddate AS DATE NO-UNDO.
    INPUT FROM VALUE(SEARCH("r-date.dat")).
    REPEAT:
    SET cnum cdate.
    ASSIGN
    imon = INTEGER(SUBSTR(cdate,1,2))
    iday = INTEGER(SUBSTR(cdate,4,2))
    iyr = INTEGER(SUBSTR(cdate,7,2))
    /* Works for years within 50 of 2000 */
    iyr = iyr + (IF (iyr < 50) THEN 2000 ELSE 1900)
    ddate = DATE(imon,iday,iyr).
    DISPLAY ddate.
    END.
    INPUT CLOSE.
    ```

24. `DATETIME` function

    Converts date and time values, or a character string, into a DATETIME value.

    将日期和时间值或字符串转换为DATETIME值。

    ```
    DEFINE VARIABLE my-datetime AS DATETIME NO-UNDO.
    /* This statement is equivalent to "my-datetime = NOW". */
    my-datetime = DATETIME(TODAY, MTIME).
    disp my-datetime.

    my-datetime = DATETIME(5, 5, 2002, 7, 15, 3).
    disp my-datetime.

    my-datetime = DATETIME("05-05-2002 07:15:03").
    disp my-datetime.
    ```


25. `DAY` function

    Evaluates a date expression and returns a day of the month as an INTEGER value from 1 to 31, inclusive.

    计算一个日期表达式并返回一个从1到31(包括31)的整数值。

    ```
    DEFINE VARIABLE d1 AS DATE NO-UNDO LABEL "Date".
    DEFINE VARIABLE d2 AS DATE NO-UNDO LABEL "Same date next year".
    DEFINE VARIABLE d-day AS INTEGER NO-UNDO.
    DEFINE VARIABLE d-mon AS INTEGER NO-UNDO.
    REPEAT:
    SET d1.
    d-day = DAY(d1).
    d-mon = MONTH(d1).
    IF d-mon = 2 AND d-day = 29 THEN d-day = 28.
    d2 = DATE(d-mon,d-day,YEAR(d1) + 1).
    DISPLAY d2.
    END.
    ```

26. `DBNAME` function
    ```
    DEFINE VARIABLE pageno AS INTEGER NO-UNDO FORMAT "zzz9" INITIAL 1.
    FORM HEADER "Date:" TO 10 TODAY
    "Page:" AT 65 pageno SKIP
    "Database:" TO 10 DBNAME FORMAT "x(60)" SKIP
    "Userid:" TO 10 USERID WITH NO-BOX NO-LABELS.
    VIEW.
    ```

27. `DECIMAL` function

    Converts an expression of any data type, with the exception of BLOB, CLOB, and RAW, to a DECIMAL value.

    将任何数据类型的表达式(BLOB、CLOB和RAW除外)转换为DECIMAL值。

    ```
    DEFINE VARIABLE new-max AS CHARACTER NO-UNDO FORMAT "x(10)".

    REPEAT:
        PROMPT-FOR Customer.CustNum WITH FRAME credit.
        FIND Customer USING Customer.CustNum.
        DISPLAY Customer.CustNum Customer.Name Customer.CreditLimit 
            WITH FRAME credit DOWN.
        DISPLAY "Enter one of:" SKIP(1)
            "a = 5000" SKIP
            "b = 2000" SKIP
            "RETURN = 1000"
            "A dollar value"
            WITH FRAME vals COLUMN 60.
        SET new-max WITH FRAME credit.
            IF new-max = "a" THEN Customer.CreditLimit = 5000.
        ELSE IF new-max = "b" THEN Customer.CreditLimit = 2000.
        ELSE IF new-max > "0" AND new-max < "999,999.99" THEN
            Customer.CreditLimit = DECIMAL(new-max).
        ELSE Customer.CreditLimit = 1000.
        DISPLAY Customer.CreditLimit WITH FRAME credit.
    END.
    ```

28. `DEFINE VARIABLE` statement

    Defines a variable for use in one or more procedures, a variable data member of a class for use in a single class or class hierarchy, or by other classes and procedures, or a variable data element for use within a single class-based method.

    定义一个变量，用于一个或多个过程;定义一个类的变量数据成员，用于单个类或类层次结构，或由其他类和过程使用;定义一个变量数据元素，用于单个基于类的方法。

29. `DEFINE STREAM` statement    ======
    
    Defines a stream for use in one or more procedures, or within a single class.  Use this statement when you want to use streams other than the two ABL built-in unnamed streams.  Using additional streams allows you to get input from more than one source simultaneously or to send output to more than one destination simultaneously.
   
   定义在一个或多个过程中或在单个类中使用的流。当您希望使用两个ABL内置未命名流以外的流时，请使用此语句。使用额外的流允许您同时从多个源获取输入，或者同时将输出发送到多个目的地。

30. `DEFINE BUFFER` statement

    ABL provides you with one default buffer for each table or temp-table that you use in a procedure or class.  ABL uses that buffer to store one record at a time from the table as the records are needed during the procedure or class.  If you need more than one record or buffer at a time for a table, you can use this statement to define alternate buffers that are created at compile time for use in one or more procedures, or within a single class or class hierarchy.

    ABL为您在过程或类中使用的每个表或临时表提供一个默认缓冲区。ABL使用该缓冲区来存储表中每次需要的一条记录，因为在过程或类中需要这些记录。如果一个表一次需要多个记录或缓冲区，则可以使用此语句来定义在编译时创建的备用缓冲区，以便在一个或多个过程中，或在单个类或类层次结构中使用。

    ```
    DEFINE BUFFER other-cust FOR Customer.
    FORM Customer WITH FRAME cre-cust.
    ON LEAVE OF Customer.PostalCode DO:
    FIND FIRST other-cust 
    WHERE other-cust.PostalCode = Customer.PostalCode:SCREEN-VALUE 
    AND other-cust.CustNum <> Customer.CustNum NO-ERROR.
    IF AVAILABLE other-cust THEN
    DISPLAY other-cust.City @ Customer.City
    other-cust.State @ Customer.State 
    other-cust.Country @ Customer.Country
    WITH FRAME cre-cust. 
    ENABLE Customer.City Customer.State Customer.Country WITH FRAME cre-cust.
    END.
    CREATE Customer.
    UPDATE Customer EXCEPT Customer.City Customer.State Customer.Country
    WITH FRAME cre-cust.
    ```

31. `DEFINE FRAME` statement

    Defines and creates a frame or dialog box that is created at compile time for use in one or more procedures, or within a single class.

    定义并创建在编译时创建的框架或对话框，以便在一个或多个过程中或在单个类中使用。

    ```
    DEFINE VARIABLE bal-avail NO-UNDO LIKE Customer.Balance
      COLUMN-LABEL "Available!Credit" NO-UNDO.
    DEFINE FRAME cust-bal 
      Customer.CustNum
      Customer.Name FORMAT "X(20)"
      Customer.CreditLimit LABEL "Limit"
      Customer.Balance
      bal-avail
      WITH CENTERED ROW 3 TITLE "Available Customer Credit" USE-TEXT.
    FOR EACH Customer NO-LOCK WITH FRAME cust-bal:
      DISPLAY 
        Customer.CustNum
        Customer.Name
        Customer.CreditLimit
        Customer.Balance
        Customer.CreditLimit - Customer.Balance @ bal-avail.
    END.
    ```

    ```
    DEFINE BUTTON b_dtl LABEL "Detail".
    DEFINE BUTTON b_next LABEL "Next".
    DEFINE BUTTON b_quit LABEL "Quit" AUTO-ENDKEY.


    DEFINE FRAME cust-info 
    Customer.CustNum
    Customer.Name FORMAT "X(20)"
    Customer.Phone
    WITH CENTERED ROW 4.

    DEFINE FRAME cust-dtl
    Customer.Address Customer.Address2  Customer.EmailAddress Customer.PostalCode
    WITH CENTERED SIDE-LABELS ROW 9.

    DEFINE FRAME butt-frame
        b_dtl b_next b_quit
        WITH ROW 1.

    ON CHOOSE OF b_dtl
        DISPLAY Customer.Address Customer.Address2  Customer.EmailAddress Customer.PostalCode
        WITH FRAME cust-dtl.

    ON CHOOSE OF b_next DO:
        HIDE FRAME cust-dtl.
        FIND NEXT Customer NO-LOCK NO-ERROR.
        IF NOT AVAILABLE Customer THEN
            FIND LAST Customer NO-LOCK.
            
        DISPLAY Customer.CustNum Customer.Name Customer.Phone
            WITH FRAME cust-info.
    END.

    ENABLE ALL WITH FRAME butt-frame.

    APPLY "CHOOSE" TO b_next IN FRAME butt-frame.
    WAIT-FOR CHOOSE OF b_quit. 
    ```

32. `DEFINE TEMP-TABLE` statement

    创建临时表

33. `DEFINE PARAMETER` statement
    
    参数定义

34. `DELETE` statement
    
    从记录缓冲区和数据库中删除一条记录。

35. `DELIMITER` option

36. `DESCENDING` option 
    
    按照降序对记录进行排序

37. `DISABLE` statement

38. `DISCONNECT` statement

    Disconnects the specified database.

    断开指定数据库的连接。
    ```
    DISCONNECT mydb.
    ```
39. `DISPLAY` statement

    输出显示

40. `DO` statement

    Groups statements into a single block, optionally specifying processing services or block properties. Use an END statement to end a DO block.

    将语句分组到单个块中，可以选择指定处理服务或块属性。使用END语句结束DO块。

    ```
    FOR EACH Customer NO-LOCK:  
    DISPLAY Customer.Name Customer.CreditLimit.  
    PAUSE 3.  
    IF Customer.CreditLimit > 80000 THEN DO:    
        Customer.CreditLimit = 80000.    
        DISPLAY Customer.Name Customer.CreditLimit.  
     END.
    END.
    ```

41. `DOWN` statement

    Positions the cursor on a new line in a down or multi-line frame.

42. `IF...THEN...ELSE` statement
    ```
    DEFINE VARIABLE ans AS LOGICAL NO-UNDO.
    DEFINE STREAM due.
    OUTPUT STREAM due TO E:/p/ovrdue.lst.
    DISPLAY STREAM due
        "Orders shipped but still unpaid as of" TODAY SKIP(2)
        WITH NO-BOX NO-LABELS CENTERED FRAME hdr PAGE-TOP.

    FOR EACH Order WITH FRAME oinfo:
        FIND Customer OF Order NO-LOCK.
        DISPLAY Order.OrderNum Customer.Name Order.OrderDate Order.PromiseDate
            Order.ShipDate.
        IF Order.ShipDate = ? THEN DO:
            IF Order.PromiseDate = ? THEN DO:
                MESSAGE "Please update the promise date.".
                REPEAT WHILE promiseDate = ?:
                    UPDATE  Order.PromiseDate WITH FRAME oinfo.
                END.
            END.
            ans = FALSE.
            MESSAGE "Has this order been shipped?" UPDATE ans.  
            IF ans THEN
            REPEAT WHILE Order.ShipDate = ?:
                UPDATE Order.ShipDate WITH FRAME oinfo.
            END.
        END.    
        ELSE DO:
            ans = TRUE.
            MESSAGE "Has this order been paid?" UPDATE ans.
            IF NOT ans THEN DO:
                DISPLAY STREAM due Order.OrderNum TO 14 Customer.Name AT 18
                    Order.OrderDate AT 42 Order.ShipDate AT 54
                    WITH NO-BOX DOWN FRAME unpaid.
            END.
        END.
    END.
    OUTPUT STREAM due CLOSE.  
    ```
43. `EMPTY TEMP-TABLE` statement
    ```
    EMPTY TEMP-TABLE temp-table-name
    ```
44. `ENCODE` function
    ```
    DEFINE VARIABLE password      AS CHARACTER NO-UNDO FORMAT "x(16)".
    DEFINE VARIABLE id            AS CHARACTER NO-UNDO FORMAT "x(12)".
    DEFINE VARIABLE n-coded-p-wrd AS CHARACTER NO-UNDO FORMAT "x(16)". 

    SET id LABEL "Enter user id" password LABEL  
        "Enter password" BLANK WITH CENTERED SIDE-LABELS. 
        
    n-coded-p-wrd = ENCODE(password). 

    DISPLAY n-coded-p-wrd LABEL "Encoded password".
    ```

45. `ENTRY` function
    
    Returns a character string (CHARACTER or LONGCHAR) entry from a list based on an integer position.

    从基于整数位置的列表中返回一个字符串(character或LONGCHAR)条目。

    ```
    DEFINE VARIABLE datein AS DATE NO-UNDO.
    DEFINE VARIABLE daynum AS INTEGER NO-UNDO.
    DEFINE VARIABLE daynam AS CHARACTER NO-UNDO INITIAL "Sunday,
    Monday, Tuesday, Wednesday, Thursday, Friday, Saturday".

    SET datein LABEL "Enter a date (mm/dd/yy)".

    daynum = WEEKDAY(datein).
    DISPLAY ENTRY(daynum,daynam) FORMAT "x(9)" LABEL "is a" WITH SIDE-LABELS.
    ```

    ```
    DEFINE VARIABLE typeface AS CHARACTER NO-UNDO.
    typeface = "-adobe-helvetica-bold-r-normal--*-210-*-*-*-*-iso*-*".
    DISPLAY ENTRY(3, typeface, "-") FORMAT "x(16)".
    ```

46. `ETIME` function

    Returns, as an INT64 value, the time (in milliseconds) elapsed since the ABL session began or since ETIME (elapsed time) was last set to 0.  To set ETIME to 0, pass it a positive logical value, such as YES or TRUE.

    作为INT64值返回自ABL会话开始或自ETIME(运行时间)最后设置为0以来所经过的时间(以毫秒为单位)。要将ETIME设置为0，需要向其传递一个正的逻辑值，如YES或TRUE。
    
    ```
    DISPLAY ETIME.
    ```

    ```
    DEFINE VARIABLE a AS INT64 NO-UNDO.
    DO:
      a = ETIME(yes).
      RUN applhelp.p.
      DISPLAY ETIME.
    END.
    ```

47. `EXCLUSIVE-LOCK` option

    独占锁

48. `EXPORT` statement
    
    Converts data to a standard character format and displays it to the current output destination (except when the current output destination is the screen) or to a named output stream. You can use data exported to a file in standard format as input to other ABL procedures.

    将数据转换为标准字符格式，并将其显示到当前输出目的地(除非当前输出目的地是屏幕)或指定的输出流。您可以使用以标准格式导出到文件的数据作为其他ABL过程的输入。

    ```
    OUTPUT TO E:/p/stream/customer.d.
    FOR EACH Customer NO-LOCK:
    EXPORT Customer.
    END.
    OUTPUT CLOSE.
    ```

    ```
    OUTPUT TO E:/p/stream/custdump.
    FOR EACH Customer NO-LOCK:
    EXPORT Customer.CustNum Customer.Name Customer.CreditLimit.
    END.
    OUTPUT CLOSE.
    ```

49. `EXTENT` option

    ```
    DEFINE VARIABLE array-var AS CHARACTER NO-UNDO EXTENT 3   INITIAL ["Add","Delete","Update"].
    ```

50. `FILL` function
    
    Generates a character string made up of a character string that is repeated a specified number of times.

    生成由重复指定次数的字符串组成的字符串。

    语法:
    ```
    FILL ( expression , repeats )
    ```
    ```
    DEFINE VARIABLE fillchar AS CHARACTER NO-UNDO FORMAT "x" INITIAL "*".
    DEFINE VARIABLE percentg AS INTEGER NO-UNDO FORMAT ">>9".
    FOR EACH Customer NO-LOCK:
    ACCUMULATE Customer.Balance (TOTAL).
    END.
    DISPLAY "Percentage of Outstanding Balance" WITH CENTERED NO-BOX.
    FOR EACH Customer NO-LOCK WHERE Customer.Balance > 0:
    percentg = Customer.Balance / (ACCUM TOTAL Customer.Balance) * 100.
    FORM SKIP Customer.Name percentg LABEL "%" bar AS CHARACTER
    LABEL " 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17"
    FORMAT "x(50)" WITH NO-BOX NO-UNDERLINE USE-TEXT.
    COLOR DISPLAY BRIGHT-RED bar.
    DISPLAY Customer.Name percentg FILL(fillchar,percentg * 3) @ bar.
    END.
    ```
51. `FIND` statement

    Locates a single record in a table and moves that record into a record buffer.

    定位表中的单个记录并将该记录移动到记录缓冲区中。

    ```
    FIND Customer 1.

    FIND Customer WHERE Customer.CustNum = 1.
    ```

52. `FOR` statement

    Starts an iterating block that reads a record from each of one or more tables at the start of each block iteration.  Use an END statement to end a FOR block.

    启动一个迭代块，在每个块迭代开始时从一个或多个表中读取一条记录。使用END语句结束FOR块。

    ```
    FOR FIRST Customer NO-LOCK BY Customer.CreditLimit:  DISPLAY Customer.END.
    ```

    ```
    FOR EACH Customer NO-LOCK BY Customer.CreditLimit:  DISPLAY Customer.  LEAVE.END.
    ```
    
53. `FORM` statement
    ```
    REPEAT FOR Customer:
      FORM 
        Customer.Name COLON 10 Customer.Phone COLON 50
        Customer.Address COLON 10 Customer.SalesRep COLON 50 SKIP
        Customer.City COLON 10 NO-LABEL Customer.State NO-LABEL 
        Customer.PostalCode NO-LABEL
        WITH SIDE-LABELS 1 DOWN CENTERED.
        PROMPT-FOR Customer.CustNum WITH FRAME cnum SIDE-LABELS CENTERED.
      FIND Customer USING Customer.CustNum.
      UPDATE Customer.Name Customer.Address Customer.City Customer.State
      Customer.PostalCode Customer.Phone Customer.SalesRep.
    END.
    ```

54. `FORMAT` option

    Defines or declares a prototype for a user-defined function, or declares a Web service operation. The following syntax boxes describe the syntax for each use of the statement, beginning with a user-defined function definition.

    定义或声明用户定义函数的原型，或声明Web服务操作。下面的语法框描述了语句的每种用法的语法，从用户定义的函数定义开始。

    ```
    /* r-udf1.p */
    /* Defines and references a user-defined function */
    /* Define doubler() */
    FUNCTION doubler RETURNS INTEGER (INPUT parm1 AS INTEGER):
    RETURN (2 * parm1). 
    END FUNCTION.
    /* Reference doubler() */
    DISPLAY "doubler(0)=" doubler(0) SKIP
    "doubler(1)=" doubler(1) skip
    "doubler(2)=" doubler(2) skip.
    ```

55. `HIDE` statement
    ```
    DEFINE VARIABLE selection AS INTEGER NO-UNDO FORMAT "9".
    FORM
    "Please Make A Selection:" SKIP(2)
    " 1. Hide Frame A. " SKIP
    " 2. Hide Frame B. " SKIP
    " 3. Hide All. " SKIP
    " 4. Hide This Frame " SKIP
    " 5. Exit " SKIP(2)
    WITH FRAME X NO-LABELS.
    REPEAT:
    VIEW FRAME x.
    DISPLAY "This is frame A."
    WITH FRAME a ROW 1 COLUMN 60.
    DISPLAY "This is frame B."
    WITH FRAME b ROW 16 COLUMN 10 4 DOWN.
    MESSAGE "Make your selection!".
    UPDATE "Selection: " selection VALIDATE(0 < selection AND selection < 7,
    "Invalid selection") AUTO-RETURN
    WITH FRAME x.
    IF selection = 1 THEN HIDE FRAME a.
    ELSE IF selection = 2 THEN HIDE FRAME b.
    ELSE IF selection = 3 THEN HIDE ALL.
    ELSE IF selection = 4 THEN HIDE FRAME x.
    ELSE IF selection = 5 THEN LEAVE.
    PAUSE.
    END.
    ```

56. `IMPORT` statement ======

57. `INDEX` function

    Returns an INTEGER value that indicates the position of the target string within the source string.

    返回一个INTEGER值，该值指示目标字符串在源字符串中的位置。
    ```
    DEFINE VARIABLE x AS CHARACTER NO-UNDO FORMAT "9"
    LABEL "Enter a digit between 1 and 5".
    DEFINE VARIABLE show AS CHARACTER NO-UNDO FORMAT "x(5)" EXTENT 5 
    LABEL "Literal" INITIAL ["One", "Two", "Three", "Four", "Five"].
    REPEAT:
    SET x AUTO-RETURN.
    IF INDEX("12345",x) = 0 THEN DO:
    MESSAGE "Digit must be 1,2,3,4, or 5. Try again.".
    UNDO, RETRY.
    END.
    ELSE DISPLAY show[INTEGER(x)].
    END.
    ```

    ```
    DEFINE VARIABLE positions AS CHARACTER NO-UNDO FORMAT "x(60)".
    DEFINE VARIABLE sentence AS CHARACTER NO-UNDO FORMAT "x(72)".
    DEFINE VARIABLE vowel AS CHARACTER NO-UNDO FORMAT "x".
    DEFINE VARIABLE found AS INTEGER NO-UNDO.
    DEFINE VARIABLE loop AS INTEGER NO-UNDO.
    DEFINE VARIABLE start AS INTEGER NO-UNDO.
    FORM sentence LABEL "Type in a sentence"
    WITH FRAME top
    TITLE "This program will tell where the vowels are in a sentence.".
    SET sentence WITH FRAME top.
    DO loop = 1 TO 5:
    ASSIGN
    positions = ""
    vowel = SUBSTRING("aeiou",loop,1)
    start = 1
    found = INDEX(sentence,vowel,start).
    DO WHILE found > 0:
    ASSIGN
    positions = positions + STRING(found) + " "
    start = found + 1
    found = INDEX(sentence,vowel,start).
    END.
    DISPLAY vowel LABEL "Vowel" positions LABEL "Is found at locations..." 
    WITH 5 DOWN.
    DOWN.
    END.
    ```

58.  `INPUT CLOSE` statement

      ```
      INPUT FROM VALUE(SEARCH("E:/p/source/r-in.dat")).
      REPEAT:
      PROMPT-FOR Customer.CustNum Customer.CreditLimit.
      FIND Customer USING INPUT Customer.CustNum.
      ASSIGN Customer.CreditLimit.
      END.
      INPUT CLOSE.
      ```

59. `INPUT FROM` statement  如例57 


60. `INPUT` function
    ```
    FOR EACH Customer:
    DISPLAY Customer.CustNum Customer.Name Customer.CreditLimit 
    LABEL "Current credit limit"
    WITH FRAME a 1 DOWN ROW 1.
    PROMPT-FOR Customer.CreditLimit LABEL "New credit limit" 
    WITH SIDE-LABELS NO-BOX ROW 10 FRAME b.
    IF INPUT FRAME b Customer.CreditLimit <> Customer.CreditLimit THEN DO:
    DISPLAY "Changing max credit of" Customer.Name SKIP
    "from" Customer.CreditLimit "to" INPUT FRAME b Customer.CreditLimit
    WITH FRAME c ROW 15 NO-LABELS.
    Customer.CreditLimit = INPUT FRAME b Customer.CreditLimit.
    END.
    ELSE DISPLAY "No change in credit limit" WITH FRAME d ROW 15.
    END.
    ```
61. `INT64` function 

62. `INTEGER` function
    
    Converts an expression of any data type, with the exception of BLOB, CLOB, and RAW, to a 32-bit integer value of data type INTEGER, rounding that value if necessary.

    将任何数据类型的表达式(BLOB、CLOB和RAW除外)转换为数据类型integer的32位整数值，必要时将该值四舍五入。

    ```
    DEFINE VARIABLE street-number AS INTEGER NO-UNDO LABEL "Street Number".
    FOR EACH Customer NO-LOCK:
    ASSIGN street-number = INTEGER(ENTRY(1, Customer.Address, " ")) NO-ERROR.
    IF ERROR-STATUS:ERROR THEN 
    MESSAGE "Could not get street number of" Customer.Address.
    ELSE 
    DISPLAY Customer.CustNum Customer.Address street-number.
    END.
    ```

63. `INTERVAL` function
    
    Returns the time interval between two DATE, DATETIME, or DATETIME-TZ values as an INT64 value. 

    Syntax:
    ```
    INTERVAL ( datetime1 , datetime2 , interval-unit )
    ```

64. `KEYCODE` function

    Evaluates a key label (such as F1) for a key in the predefined set of keyboard keys and returns the corresponding = key code (such as 301) as an INTEGER value. See OpenEdge Development: Programming Interfaces for a list of key codes and key labels.

    在预定义的键盘键集中计算键标签(如F1)，并返回相应的=键代码(如301)作为INTEGER值。请参阅OpenEdge开发:编程接口以获取密钥代码和密钥标签的列表。

    Syntax 
    ```
    KEYCODE ( key-label )
    ```

    ```
    DEFINE VARIABLE msg AS CHARACTER NO-UNDO EXTENT 3.
    DEFINE VARIABLE ix AS INTEGER NO-UNDO INITIAL 1.
    DEFINE VARIABLE newi AS INTEGER NO-UNDO INITIAL 1.
    DISPLAY 
    " Please choose " SKIP(1)
    " 1 Run order entry " @ msg[1]
    ATTR-SPACE SKIP
    " 2 Run receivables " @ msg[2]
    ATTR-SPACE SKIP
    " 3 Exit " @ msg[3]
    ATTR-SPACE SKIP
    WITH CENTERED FRAME menu NO-LABELS.
    REPEAT:
    COLOR DISPLAY MESSAGES msg[ix] WITH FRAME menu.
    READKEY.
    IF LASTKEY = KEYCODE("CURSOR-DOWN") AND ix < 3 THEN
    newi = ix + 1.
    ELSE IF LASTKEY = KEYCODE("CURSOR-UP") AND ix > 1 THEN
    newi = ix - 1.
    ELSE IF LASTKEY = KEYCODE("GO") OR LASTKEY = KEYCODE("RETURN") THEN LEAVE.
    IF ix <> newi THEN 
    COLOR DISPLAY NORMAL msg[ix] WITH FRAME menu.
    ix = newi.
    END.
    ```

65. `KEYFUNCTION` function

    Evaluates an integer expression (such as 301) and returns a character string that is the function of the key associated with that integer expression (such as GO).

    计算一个整数表达式(如301)并返回一个字符串，该字符串是与该整数表达式(如GO)相关联的键的函数。



    ```
    DEFINE VARIABLE msg AS CHARACTER NO-UNDO EXTENT 3.
    DEFINE VARIABLE ix AS INTEGER NO-UNDO INITIAL 1.
    DEFINE VARIABLE newi AS INTEGER NO-UNDO INITIAL 1.
    DEFINE VARIABLE func AS CHARACTER NO-UNDO.
    DISPLAY 
    " Please choose " SKIP(1)
    " 1 Run order entry " @ msg[1] ATTR-SPACE SKIP
    " 2 Run receivables " @ msg[2] ATTR-SPACE SKIP
    " 3 Exit " @ msg[3] ATTR-SPACE SKIP
    WITH CENTERED FRAME menu NO-LABELS.
    REPEAT:
    COLOR DISPLAY MESSAGES msg[ix] WITH FRAME menu.
    READKEY.
    func = KEYFUNCTION(LASTKEY).
    IF func = "CURSOR-DOWN" AND ix < 3 THEN 
    newi = ix + 1.
    ELSE IF func = "CURSOR-UP" AND ix > 1 THEN
    newi = ix - 1.
    ELSE IF func = "GO" OR func = "RETURN" THEN LEAVE.
    IF ix <> newi THEN 
    COLOR DISPLAY NORMAL msg[ix] WITH FRAME menu.
    ix = newi.
    END.
    ```

66. `KEYLABEL` function

    Evaluates a key code (such as 301) and returns a character string that is the predefined keyboard label for that key (such as F1).

    计算一个键代码(如301)并返回一个字符串，该字符串是该键的预定义键盘标签(如F1)。

    ```
    DISPLAY "Press the " + KBLABEL("GO") + " key to leave procedure"
    FORMAT "x(50)".
    REPEAT:
    READKEY.
    HIDE MESSAGE.
    IF LASTKEY = KEYCODE(KBLABEL("GO")) THEN RETURN.
    MESSAGE "Sorry, you pressed the" KEYLABEL(LASTKEY) "key.".
    END.
    ```

67. `LAST` function
    Syntax 
    ```
    LAST ( break-group )
    ```

    ```
    FOR EACH Item NO-LOCK BY Item.OnHand * Item.Price DESCENDING:
    DISPLAY Item.ItemNum Item.OnHand * Item.Price (TOTAL) LABEL "Value-oh"
    WITH USE-TEXT.
    END.
    FOR EACH Item NO-LOCK BREAK BY Item.OnHand * Item.Price DESCENDING:
    FORM Item.ItemNum value-oh AS DECIMAL LABEL "Value-oh" 
    WITH COLUMN 40 USE-TEXT.
    DISPLAY Item.ItemNum Item.OnHand * Item.Price @ value-oh.
    ACCUMULATE Item.OnHand * Item.Price (TOTAL).
    IF LAST(Item.OnHand * Item.Price) THEN DO:
    UNDERLINE value-oh.
    DISPLAY ACCUM TOTAL Item.OnHand * Item.Price @ value-oh.
    END.
    END.
    ```

68. `LC` function

    Converts any uppercase characters in a CHARACTER or LONGCHAR expression to lowercase characters, and returns the result.

    将CHARACTER或LONGCHAR表达式中的任何大写字符转换为小写字符，并返回结果。

    ```
    REPEAT:
    PROMPT-FOR Customer.CustNum.
    FIND Customer USING Customer.CustNum.
    DISPLAY Customer.Name.
    UPDATE Customer.SalesRep.
    Customer.SalesRep = CAPS(SUBSTRING(Customer.SalesRep, 1, 1) ) +
    LC(SUBSTRING(Customer.SalesRep, 2) ).
    DISPLAY Customer.SalesRep.
    END.
    ```

69. `LDBNAME` function

    Returns the logical name of a database that is currently connected.

    返回当前连接的数据库的逻辑名称。

    ```
    DEFINE VARIABLE testnm AS CHARACTER NO-UNDO.
    SET testnm.
    IF LDBNAME(testnm) = testnm THEN 
    MESSAGE testnm "is a true logical database name.".
    ELSE IF LDBNAME(testnm) = ? THEN 
    MESSAGE testnm "is not the name or alias of any connected database.".
    ELSE 
    MESSAGE testnm "is an ALIAS for database " LDBNAME(testnm).
    ```

70. `LEAVE` statement
    
    Exits from a block.  Execution continues with the first statement after the end of the block.

    退出一个块。继续执行结束后的第一个语句。

    ```
    DEFINE VARIABLE valid-choice AS CHARACTER NO-UNDO INITIAL "NPFQ".
    DEFINE VARIABLE selection AS CHARACTER NO-UNDO FORMAT "x".
    main-loop:
    REPEAT:
    choose:
    REPEAT ON ENDKEY UNDO choose, RETURN:
    MESSAGE "(N)ext (P)rev (F)ind (Q)uit"
    UPDATE selection AUTO-RETURN.
    /* Selection was valid */
    IF INDEX(valid-choice, selection) <> 0 THEN LEAVE choose.
    BELL.
    END. /* choose */
    /* Processing for menu choices N, P, F here */
    IF selection = "Q" THEN LEAVE main-loop.
    END. /* REPEAT */
    ```

71. `LEFT-TRIM` function

    Removes leading white space, or other specified characters, from a CHARACTER or LONGCHAR expression.

    从CHARACTER或LONGCHAR表达式中删除前导空格或其他指定字符。

    ```
    DEFINE VARIABLE txt AS CHARACTER NO-UNDO FORMAT "X(26)"   INITIAL "***** This is a test *****".

    DISPLAY LEFT-TRIM(txt,"* ") FORMAT "x(26)". 
    ```

72. `LENGTH` statement

    Changes the number of bytes in a raw variable.

    更改原始变量中的字节数。

    ```
    /* You must connect to a non-OpenEdge demo database to run this procedure */
    DEFINE VARIABLE r1 as RAW NO-UNDO.
    FIND Customer NO-LOCK WHERE Customer.CustNum = 29.
    r1 = RAW(Customer.Name).
    LENGTH(r1) = 2.
    ```

73. `LOCKED` function

    Returns a TRUE value if a record is not available to a prior FIND . . . NO-WAIT statement because another user has locked a record.

    ```
    REPEAT:
    PROMPT-FOR Customer.CustNum.
    FIND Customer USING Customer.CustNum NO-ERROR NO-WAIT.
    IF NOT AVAILABLE Customer THEN DO:
    IF LOCKED Customer THEN 
    MESSAGE "Customer record is locked".
    ELSE 
    MESSAGE "Customer record was not found".
    NEXT.
    END.
    DISPLAY Customer.CustNum Customer.Name Customer.City Customer.State.
    END.
    ```

74. `LOGICAL` function

    Converts any data type into the LOGICAL data type.

    ```
    DEFINE VARIABLE mychar AS CHARACTER NO-UNDO. 
    DEFINE VARIABLE v-log  AS LOGICAL   NO-UNDO.

    mychar = "sii".

    v-log = LOGICAL(mychar, "sii/no").
    DISPLAY v-log.
    ```
    
75. `LOOKUP` function

    Returns an INTEGER value giving the position of an expression in a list. Returns a 0 if the expression is not in the list.

    ```
    DEFINE VARIABLE stlist AS CHARACTER NO-UNDO
    INITIAL "ME,MA,VT,RI,CT,NH".
    DEFINE VARIABLE state AS CHARACTER NO-UNDO FORMAT "x(2)".
    REPEAT:
    SET state LABEL "Enter a New England state, 2 characters".
    IF LOOKUP(state, stlist) = 0 THEN 
    MESSAGE "This is not a New England state".
    END.
    ```

76. `MATCHES` operator

    ```
    FOR EACH Customer NO-LOCK WHERE Customer.Address MATCHES("*St"):
    DISPLAY Customer.Name Customer.Address Customer.City Customer.State
    Customer.Country.
    END.
    ```
77. `MAXIMUM` function

    Compares two or more values and returns the largest value.

    ```
    DEFINE VARIABLE cred-lim2 AS DECIMAL NO-UNDO FORMAT ">>,>>9.99".
    FOR EACH Customer NO-LOCK:
    cred-lim2 = IF Customer.CreditLimit < 20000 THEN
    Customer.CreditLimit + 10000 ELSE 30000.
    DISPLAY Customer.CreditLimit cred-lim2
    MAXIMUM(cred-lim2, Customer.CreditLimit) 
    LABEL "Maximum of these two values".
    END.
    ```

78. `MESSAGE` statement
    ```
    REPEAT:
    PROMPT-FOR Customer.CustNum.
    FIND Customer USING Customer.CustNum NO-ERROR.
    IF NOT AVAILABLE Customer THEN DO:
    MESSAGE "Customer with CustNum " INPUT Customer.CustNum
    " does not exist. Please try another".
    UNDO, RETRY.
    END.
    ELSE
    DISPLAY Customer.Name Customer.SalesRep.
    END.
    ```

79. `MINIMUM` function

    Compares two or more values and returns the smallest.

80. `MODULO` operator

    求余

    ```
    REPEAT:
    SET qty-avail AS INTEGER LABEL "Qty. Avail.".
    SET std-cap AS INTEGER LABEL "Std. Truck Capacity".
    DISPLAY TRUNCATE(qty-avail / std-cap,0) FORMAT ">,>>9" LABEL "# Full Loads"
    qty-avail MODULO std-cap LABEL "Qty. Left".
    END.
    ```

81. `MONTH` function

    Evaluates a date expression and returns a month INTEGER value from 1 to 12, inclusive.

    ```
    FOR EACH Order NO-LOCK:
    IF (MONTH(Order.PromiseDate) < MONTH(TODAY) OR
    YEAR(Order.PromiseDate) < YEAR(TODAY)) AND Order.ShipDate = ? THEN 
    DISPLAY Order.OrderNum LABEL "Order Num" Order.PO LABEL "P.O. Num"
    Order.PromiseDate LABEL "Promised By"
    Order.OrderDate LABEL "Ordered" terms
    WITH TITLE "These orders are overdue".
    END.
    ```

82. `MTIME` function

    Returns an INTEGER value representing the time in milliseconds. If the MTIME function has no arguments, it returns the current number of milliseconds since midnight (similar to TIME, which returns seconds since midnight).

    返回以毫秒为单位表示时间的INTEGER值。如果MTIME函数没有参数，它返回从午夜开始的当前毫秒数(类似于TIME，它返回从午夜开始的秒数)。

83. `NEW SHARED` option

84. `NEXT` statement

    Goes directly to the END of an iterating block and starts the next iteration of the block.

    直接转到迭代块的END，并开始该块的下一次迭代。

    ```
    PROMPT-FOR Customer.SalesRep LABEL "Enter salesman initials"
    WITH SIDE-LABELS CENTERED.
    FOR EACH Customer:
    IF Customer.SalesRep <> INPUT Customer.SalesRep THEN NEXT.
    DISPLAY Customer.CustNum Customer.Name Customer.City Customer.State 
    WITH CENTERED USE-TEXT.
    END.
    ```

85. `NEXT-PROMPT` statement 

    ```
    FOR EACH Customer:
    UPDATE Customer.Name Customer.Contact Customer.SalesRep  WITH 2 COLUMNS.
    IF Customer.Contact EQ " " THEN DO:
    MESSAGE "You must enter a contact".
    NEXT-PROMPT Customer.Contact.
    UNDO, RETRY.
    END.
    END.
    ```

86. `NEXT-VALUE` function

    Returns the next INT64 value of a static sequence, incremented by the positive or 
negative value defined in the Data Dictionary

返回静态序列的下一个INT64值，以数据字典中定义的正值或负值递增

     ```
     TRIGGER PROCEDURE FOR Create OF Item.
    /* Automatically assign a unique item number using NextItemNum seq */
    ASSIGN Item.ItemNum = NEXT-VALUE(NextItemNum).
     ```

87. `NO-MESSAGE` option

    Tells the AVM to pause but not to display a message on the status line of the terminal screen.

88. `NO-PAUSE` option

    Does not pause before clearing the frame.

89. `NOW` function

    Returns the current system date, time, and time zone as a DATETIME-TZ value.

    以DATETIME-TZ值返回当前系统日期、时间和时区。

    ```
    DEFINE VARIABLE v-datetime    AS DATETIME    NO-UNDO.
    DEFINE VARIABLE v-datetime-tz AS DATETIME-TZ NO-UNDO.

    ASSIGN
      v-datetime    = NOW
      v-datetime-tz = NOW.
    ```
90. `NO-WAIT` option

    Causes FIND to return immediately and raise an error condition if the record is locked by another user (unless you use the NO-ERROR option on the same FIND statement). For example:

    ```
    FIND Customer USING cust-name NO-ERROR NO-WAIT.
    ```

    Without the NO-WAIT option, the AVM waits until the record is available.

    -------------------------
    Specifies that the GET statement returns immediately if the record cannot be accessed because it is locked by another user.  If you do not use the NO-WAIT option, the GET statement waits until the record can be accessed.  This applies to all buffers in a join.  If you specify NO-WAIT and the record is locked by another user, the record is returned to you with NO-LOCK and the LOCKED function returns TRUE for the record.

    指定如果记录因被其他用户锁定而无法访问，则GET语句立即返回。如果不使用NO-WAIT选项，GET语句将一直等待，直到可以访问记录。这适用于连接中的所有缓冲区。如果您指定了NO-WAIT，并且该记录被其他用户锁定，则该记录将以NO-LOCK返回给您，并且locked函数将该记录返回TRUE。

91. `NUM-DBS` function

    Takes no arguments; returns the number of connected databases as an INTEGER value.
    不需要参数;以整数值形式返回已连接数据库的数目。

    ```
    DEFINE VARIABLE ix AS INTEGER NO-UNDO.
    REPEAT ix = 1 TO NUM-DBS:
    DISPLAY LDBNAME(ix) DBRESTRICTIONS(ix) FORMAT "x(40)".
    END.
    ```
92. `NUM-ENTRIES` function

    Returns the number of elements in a list of character strings as an INTEGER value.

    以INTEGER值的形式返回字符串列表中的元素个数。

    ```
    DEFINE VARIABLE ix AS INTEGER NO-UNDO.
    DEFINE VARIABLE regions AS CHARACTER NO-UNDO 
    INITIAL "Northeast,Southest,Midwest,Northwest,Southwest".
    REPEAT ix = 1 TO NUM-ENTRIES(regions):
    DISPLAY ENTRY(ix, regions) FORMAT "x(12)".
    END.
    ```
93. `OPSYS` function

    Identifies the operating system being used, so that a single version of a procedure can work differently under different operating systems. Returns the value of that operating system. Valid values are “UNIX” and “WIN32".

    标识正在使用的操作系统，以便过程的单一版本可以在不同的操作系统下以不同的方式工作。返回该操作系统的值。取值为UNIX和WIN32。

    ```
    IF OPSYS = "UNIX" THEN UNIX ls.
    ELSE IF OPSYS = "WIN32" THEN DOS dir.
    ELSE MESSAGE OPSYS "is an unsupported operating system".
    ```

94. `OS-COMMAND` statement
    Escapes to the current operating system and executes an operating system command.
    ```
    DEFINE VARIABLE comm-line AS CHARACTER NO-UNDO FORMAT "x(70)".
    REPEAT:
    UPDATE comm-line.
    OS-COMMAND VALUE(comm-line).
    END.
    ```

95. `OS-APPEND` statement   =================!!!

    Executes an operating system file append command from within ABL.

    从ABL中执行操作系统文件附加命令。


96. `OS-COPY` statement

    Executes an operating system file copy command from within ABL.
    从ABL中执行操作系统文件复制命令。

    ```
    DEFINE VARIABLE sourcefilename AS CHARACTER NO-UNDO.
    DEFINE VARIABLE copyfilename AS CHARACTER NO-UNDO FORMAT "x(20)" 
    VIEW-AS FILL-IN.
    DEFINE VARIABLE OKpressed AS LOGICAL NO-UNDO INITIAL TRUE.
    Main:
    REPEAT:
    SYSTEM-DIALOG GET-FILE sourcefilename
    TITLE "Choose File to Copy"
    MUST-EXIST
    USE-FILENAME
    UPDATE OKpressed.
    IF OKpressed = FALSE THEN
    LEAVE Main.
    UPDATE copyfilename WITH FRAME copyframe.
    OS-COPY VALUE(sourcefilename) VALUE(copyfilename).
    END.
    ```

97. `OS-CREATE-DIR` statement

    Executes an operating system command from within ABL that creates one or more new directories.

    从ABL中执行一个操作系统命令，该命令创建一个或多个新目录。

    ```
    DEFINE VARIABLE stat AS INTEGER NO-UNDO.
    DEFINE VARIABLE dir_name AS CHARACTER NO-UNDO FORMAT "x(64)"
    LABEL "Enter the name of the directory you want to create.".
    UPDATE dir_name.
    OS-CREATE-DIR VALUE(dir_name).
    stat = OS-ERROR.
    IF stat NE 0 THEN
    MESSAGE "Directory not created. System Error #" stat.
    ```
98. `OS-DELETE` statement

    Executes an operating system file or directory delete from within ABL.  Can delete one or more files, a directory, or an entire directory branch.

    从ABL中执行操作系统文件或目录删除。可以删除一个或多个文件、一个目录或整个目录分支。

    ```
    DEFINE VARIABLE filename AS CHARACTER NO-UNDO.
    DEFINE VARIABLE OKpressed AS LOGICAL NO-UNDO INITIAL TRUE.
    Main:
    REPEAT:
    SYSTEM-DIALOG GET-FILE filename
    TITLE "Choose File to Delete"
    MUST-EXIST
    USE-FILENAME
    UPDATE OKpressed.
    IF OKpressed = FALSE THEN LEAVE Main.
    ELSE OS-DELETE VALUE(filename).
    END.
    ```

99. `OS-ERROR` function

    Returns, as an INTEGER value, an ABL error code that indicates whether an execution error occurred during the last OS-APPEND, OS-COPY, OS-CREATE-DIR, OS-DELETE, OS-RENAME or SAVE CACHE statement.

    以INTEGER值返回一个ABL错误码，该错误码表示在最后一次执行OS-APPEND、OS-COPY、OS-CREATE-DIR、OS-DELETE、OS-RENAME或SAVE CACHE语句时是否发生错误。

    ```
    DEFINE VARIABLE err-status AS INTEGER NO-UNDO.
    DEFINE VARIABLE filename AS CHARACTER NO-UNDO FORMAT "x(40)"
    LABEL "Enter a file to delete".
    UPDATE filename.
    OS-DELETE VALUE(filename).
    err-status = OS-ERROR.
    IF err-status <> 0 THEN
    CASE err-status:
    WHEN 1 THEN
    MESSAGE "You are not the owner of this file or directory.".
    WHEN 2 THEN 
    MESSAGE "The file or directory you want to delete does not exist.". 
    OTHERWISE
    DISPLAY "OS Error #" + STRING(OS-ERROR,"99") FORMAT "x(13)" 
    WITH FRAME b.
    END CASE.
    ```


100. `OS-GETENV` function

      Returns a string that contains the value of the desired environment variable in the 
      environment in which the ABL session is running.
      对象中所需环境变量的值
      运行ABL会话的环境。

      ```
      DEFINE VARIABLE pathname AS CHARACTER NO-UNDO FORMAT "x(32)"
      LABEL "The report will be stored in ".
      DEFINE VARIABLE report_name AS CHARACTER NO-UNDO FORMAT "x(32)"
      LABEL "Please enter report name." .
      UPDATE report_name.
      pathname = OS-GETENV("DLC") + "/" + report_name.
      DISPLAY pathname WITH FRAME b SIDE-LABELS.
      ```
101. `OS-RENAME` statement

     Executes an operating system file rename or directory rename command from within ABL.
     从ABL中执行操作系统文件重命名或目录重命名命令。

     ```
      DEFINE VARIABLE sourcefile AS CHARACTER NO-UNDO.
      DEFINE VARIABLE targetfile AS CHARACTER NO-UNDO FORMAT "x(20)" 
      VIEW-AS FILL-IN.
      DEFINE VARIABLE OKpressed AS LOGICAL NO-UNDO INITIAL TRUE.
      Main:
      REPEAT:
      SYSTEM-DIALOG GET-FILE sourcefile
      TITLE "Choose a File or Directory to Rename"
      MUST-EXIST
      USE-FILENAME
      UPDATE OKpressed.
      IF OKpressed = FALSE THEN
      LEAVE Main.
      UPDATE targetfile WITH FRAME newnameframe.
      OS-RENAME VALUE(sourcefile) VALUE(targetfile).
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


