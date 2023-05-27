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
1. `OF`
```
FOR EACH Customer NO-LOCK WHERE Customer.State = “NH” BY  Customer.City:
  DISPLAY Customer.CustNum Customer.Name Customer.City.
  FOR EACH Order OF Customer NO-LOCK:
    DISPLAY Order.OrderNum Order.OrderDate Order.ShipDate.
  END.
END.
```
2. `WITH CENTERED`

```
FOR EACH Customer NO-LOCK WHERE Customer.State = “NH” BY  Customer.City:
  DISPLAY Customer.CustNum Customer.Name Customer.City.
  FOR EACH Order OF Customer NO-LOCK:
    DISPLAY Order.OrderNum Order.OrderDate Order.ShipDate  ` WITH CENTERED.
  END.
END.
```
3. LABEL/FORMAT
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
