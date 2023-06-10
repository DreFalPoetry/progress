# 常用语句

`absolute`
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

