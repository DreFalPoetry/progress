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



