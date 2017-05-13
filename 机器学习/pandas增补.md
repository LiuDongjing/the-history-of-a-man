# pandas增补
## concat
官方[API][concat]

把多个DataFrame在行方向上链接在一起
```python
df1=pd.DataFrame({'a':[1,2],'b':[2,4]})
df2 = pd.DataFrame({'a':[11,22],'b':[22,44]})
print(pd.concat([df1,df2], ignore_index=True))
```
输出结果
```
    a   b
0   1   2
1   2   4
2  11  22
3  22  44
```

在列方向上并在一起
```python
df3= pd.DataFrame({'c':[12,21]})
print(pd.concat([df1,df3],axis=1))
```

输出结果
```
   a  b   c
0  1  2  12
1  2  4  21
```

## to_datetime
```python
user_pay['TIME_STA'] = pd.to_datetime(user_pay['TIME_STA'])
user_pay['DATE'] = pd.to_datetime(user_pay['TIME_STA']).dt.date
user_pay['HOUR'] = pd.to_datetime(user_pay['TIME_STA']).dt.hour
```

[concat]: http://pandas.pydata.org/pandas-docs/stable/generated/pandas.concat.html#pandas.concat