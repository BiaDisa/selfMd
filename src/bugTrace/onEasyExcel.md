### 问题1：字段对不上



req: 

```````````````JSON
{
    "presentId": 1,
    "ossUrl": "https://appletsstatic.jniu.com/work-weixin/activity/d7d45da1-881d-4517-ab7d-b6848b25c0da.xlsx"
}
```````````````



bug-source:

没有使用

`````````
@ExcelProperty	
`````````

注解的字段，依然会被读到代表表头的map中进行format，和原本字段**一旦类型不匹配**，就出了bug





如果字段对不上，有时需要在不读取的字段上添加：

``````````````java
@ExcelIgnore
``````````````

注解，在

> ```
> ModelBuildEventListener
> ```

中，才能正确读取**对应字段**  lNO:123



但是在有些情形下，是可以正常运行的：



req：

``````````JSON
{
ossUrl: "https://appletsstatic.jniu.com/work-weixin/activity/b22b8dc0-a064-485a-be51-ce57605877cd.xlsx"
presentId: 2
}
``````````

原因： 时间在excel中如果格式正确，会转化成**整数类型的**，但出错的那份文件中格式错误了。



