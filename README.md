# progress

### 语法
---------------------
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

