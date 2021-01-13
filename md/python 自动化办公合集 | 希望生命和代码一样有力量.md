> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.eatqionline.top:8034](https://www.eatqionline.top:8034/article/2020/12/11/3.html)

从简化办公开始学习 Python
----------------

这篇文章整理自我以前写的一些脚本工具，可能篇幅会比较长，内容比较枯燥，大家可以按需复制。

主要包含以下五个内容：【批量改名】【转译 json】【销售预测】【表格汇总】【批量录单】

### 批量改名

可能在我们的某些工作中，会需要将收集到的资源进行改头换面 (改名)，那如果一个个的去改无疑是繁琐且浪费时间的。

在这里学会一点简单的 python，能大大简化这个过程，毕竟工具写好一劳永逸。

```
import os
import sys
#把名字中的 A 换成 B
def chang(name,A,B):
    new=name.replace(A,B)
    return new
#获取路径
#lujing = '"'+input(' 请输入要批处理修改文件名的所有文件所在路径：')+'"'
lujing =input(' 请输入要批处理修改文件名的所有文件所在路径：')
A = input(' 请输入文件名中需要批处理变动的部分：')
B = input(' 请输入想要将这部分替换为：')
path = u''+lujing

filelist = os.listdir(path)  # 该文件夹下所有的文件（包括文件夹）
for i in range(0,len(filelist)):  # 遍历所有文件
    files = str(filelist[i])
    Olddir = os.path.join(path, files)  # 原来的文件路径
    if os.path.isdir(Olddir):  # 如果是文件夹则跳过
        continue
    filename = os.path.splitext(files)[0]  # 获取文件名
    # 把列表转换成没有间隔的字符串，因为文件名要以字符串形式存在
    filenameToStr =''.join(filename)
    filename1 = chang(filenameToStr, A, B)
    filetype = os.path.splitext(files)[-1]  # 文件扩展名
    Newdir = os.path.join(path, filename1 + filetype)  # 新的文件名
    os.rename(Olddir, Newdir)  # 重命名
    sys.stdout.write('\r[ === [%d/%d] %d%% === ]' % ((i + 1),len(filelist), int((i + 1) / len(filelist) * 100)))
```

### 转译 json

做运营的朋友, 对 wetool 这款用于社群管理的工具（现在好像不能用了）, 应该并不陌生。

但是有个问题，用它导出来的好友列表，是不能直接可读的 json 文件，更不要说方便的进行筛选了。

那么这里教大家用 python 来解决这个问题。将 json 文件变成可操作性较好的表格文件。

```
#coding:utf-8
#把特定的字符串写入特定路径按特定名和格式保存
def save(who,lujing,name):
    baocunming = str(name)
    LUJING_NEW1 = lujing + '\\' + baocunming + '.xlsx'
    LUJING_NEW2 = lujing + '\\' + baocunming + '.txt'
    f = open(LUJING_NEW1, 'w',encoding='utf-8')
    f.write(who)
    f.close()
    f = open(LUJING_NEW2, 'w', encoding='utf-8')
    f.write(who)
    f.close()

#1. 提示输入要操作的文件路径及文件名
rule = input(' 请输入文件所在路径：')
name = input(' 请输入文件名（注意 本程序只可以处理 json 文件）：')

OK = rule+'\\' +name+'.json'
#C:\Users\admi\Desktop\Python_test
#黑名单
import json
with open(OK,'rb') as ok:
    A = json.load(ok)
news = ''
heard = {}
for a in A:
    for p, q in a.items():
        heard[p] = str(p)
heards = []
for p, q in heard.items():
    news = news + str(p) + '\t'
    heards.append(str(p))
news = news + '\n'
for a in A:
    one = ''
    lines = []
    for p, q in a.items():
        lines.append(str(p))
    for i in heards:
        if i not in lines:
            one = one + ''+ '\t'
        else:
            one = one + str(a[i]) + '\t'
    #print(one)
    news = news + one + '\n'
save(news,rule,name)
input(' 操作结束 回车退出 在源文件路径中查看 ')
```

### 销售预测

如果是做商品企划管理，或者销售相关岗位的朋友，可能在年度季度计划的时候，通常要进行一个复杂的计算过程：销售预测安排销售计划

这里也给大家提供一个销售预测的解决方法

```
import os                                                                                #导入 os 模块
import os.path                                                                           #导入 os.path 模块
import xlrd                                                                              #导入第三方库 xlrd
import re
import sys                                                                               #导入 sys 模块
import glob                                                                              #导入 glob 模块
#读取单个表格为字典
def read_oneExcel_in(lujing,wenjian_name,geshihouzhui,yemian=0):
    suoying = lujing + '\\' + wenjian_name + '.' + geshihouzhui
    jian = wenjian_name
    WENJIAN = xlrd.open_workbook(suoying)  # 打开文件
    YE = WENJIAN.sheets()[yemian]  # 通过索引顺序获取锁定第一页
    # 获取数据范围（行数和列数）
    Hang = YE.nrows  # 获取行数
    Lie = YE.ncols  # 获取列数
    jian = {}  # 声明存放字典的值也是字典
    # 遍历列表
    for hang in range(0, Hang):  # 遍历行数
        zhushi_hang = (' 第 ' + str(hang + 1) + ' 行 ')  # 形成行书签
        jian[zhushi_hang] = {}  # 每一行作为一个字典
       for lie in range(0, Lie):  # 遍历列数
            YE_00 = re.split(r"text:'|'", str(YE.cell(hang, lie)))  # 提取表格关键信息
            for ccc in YE_00:  # 遍历提取信息形成的过渡列表
                if ccc != '':  # 剔除列表里面的无关值
                   zhushi_lie = (' 第 ' + str(lie + 1) + ' 列 ')  # 形成列书签
                   if ccc == 'empty:':
                        ccc = ''
                   if 'number:' in ccc:
                        xxx_long = ccc.index('.')
                        ccc = int(ccc[7:xxx_long])
                    if 'xldate:' in str(ccc):
                        xxx_long_ = str(ccc).index('.')
                        ccc = int(str(ccc)[7:xxx_long_])
                    jian[zhushi_hang][zhushi_lie] = ccc
    return jian
#在单独表字典中单独读取某列为列表
def get_line_excel(biao_zidian,lie_num):
    key = []
    lie_liebiao = []
    for a in biao_zidian.keys():
        key.append(a)
    for key_i in key:
        #biao_zidian[key_i][' 第 ' + str(lie_num) + ' 列 ']
        lie_liebiao.append(str(biao_zidian[key_i][' 第 '+str(lie_num)+' 列 ']))
    return lie_liebiao
#以一定周期对列表输出移动求和的值列表
def SUM_T_yidong(Y,T):
    SUM_T = []
    for num_1 in list(range(1, len(Y) + 1 - T)):
        Sum_1 = sum(Y[num_1:num_1 + T])
        SUM_T.append(Sum_1)
    return SUM_T
#按照给定的周期，返回周期内累计求和值列表
def Fenduan_Sum(Y,T):
    Fenduan_SUM = []
    for i in list(range(0, len(Y))):
        sum_1 = Y[int(int(i / T) * T):i + 1]
        SUM = sum(sum_1)
        Fenduan_SUM.append(SUM)
    return Fenduan_SUM
#如果只有一组 Y，only_Y 来计算得到方程系数，给出需要预测的步长，如预测未来 N 个周期的 Y 值，则返回预测值列表
def oneline_huigui_only_Y_yuce(Y,yuce_after_num):
    X=list(range(1,len(Y)+1))
    X_pingjun = sum(X) / len(Y)
    Y_pingjun = sum(Y) / len(Y)
    Sum_1 = []  # 放 Xi*Yi
    Sum_2 = []  # 放 Xi*Xi
    for n in list(range(0, len(Y))):
        sum_1 = X[n] * Y[n]
        Sum_1.append(sum_1)
        sum_2 = X[n] * X[n]
        Sum_2.append(sum_2)
    b = (sum(Sum_1) - len(Y) * X_pingjun * Y_pingjun) / (sum(Sum_2) - len(Y) * X_pingjun * X_pingjun)
    a = Y_pingjun - b * X_pingjun
    #print(' 一元线性回归公式：Y='+str(b)+'X+'+str(a))
    Yuce = list(range(len(Y)+1,len(Y)+yuce_after_num+1))
    Yucezhi = []
    for yuce_X_num in Yuce:
        Y_yuce = b*yuce_X_num+a
        Yucezhi.append(Y_yuce)
    return Yucezhi
#把列表写入字典的第几行
def biao_write_zidian_hang(zidian,biao,hangshu):
    zidian[' 第 '+ str(hangshu)+ ' 行 ']={}
    for i in list(range(0,len(biao))):
        zidian[' 第 ' + str(hangshu) + ' 行 '][' 第 ' + str(i+1) + ' 列 '] = biao[i]
#把表格字典转化成表格样式的一整个字符串 返回字符串
def write_excel(A):                                                                      #创建一个转换字典为特定字符表的函数
   for B,C in A.items():                                                                #遍历表字典内的行：列键值对
       Hang = int(len(A))                                                               #获得行数
       Lie  = int(len(C))                                                               #获得列数
       Lie_Biao = []                                                                    #创建中转列：值对的列表
       Zuizhong = []                                                                    #创建最终所有列值对字符串列表
       for hhh in A.values():                                                           #遍历表中的列字典
           for lll in hhh.values():                                                     #遍历列字典的值
                Lie_Biao.append(str(lll))                                                #把值字符串化添加到值列表
       for hangshu in range(0, Hang + 1):                                               #定义行数为跳转步长
           guodu = Lie_Biao[hangshu * Lie:hangshu * Lie + Lie]                          #定义按每行转化成过渡切片
           jiange_1 = '\t'                                                              #定义一个制表符
           meiyihang = jiange_1.join(guodu)                                             #用制表符来连接切片中的每一列
            Zuizhong.append(meiyihang)                                                   #把由每一列连起来的每一行添加到最终列表
       jiange_2 = '\n'                                                                  #定义一个换行符
       zhenggebiao = jiange_2.join(Zuizhong)                                            #用换行符来连接最终列表内每个行元素
       return zhenggebiao                                                               #返回一个表示整个表的字符串
#把特定的字符串写入特定路径按特定名和格式保存
def save_excel(baocunduixiang,wenjianjialujing,baocunming,geshihouzhui):
    baocunming = str(baocunming)                                                         #指定保存名
    LUJING_NEW = wenjianjialujing + '\\' + baocunming + '.'+ geshihouzhui                #拼装最终路径
    f = open(LUJING_NEW, 'w')                                                            # 在目的路径声明一个文件及格式并打开
    f.write(baocunduixiang)                                                              # 把处理好的信息写入这个文件
    f.close()                                                                            # 关闭写入操作
#0. 导入三年的销售数据
print('——————请单列导入需要处理的数据——————')
lujing = input(' 输入数据所在文件夹路径：')
wenjian_name = input(' 输入数据所在的表格文件名：')
baocun_name = str(wenjian_name + '_预测报告 ')
geshihouzhui = input(' 输入该表格文件的格式后缀 xls/xlsx/csv：')
shuju = read_oneExcel_in(lujing,wenjian_name,geshihouzhui)#导入数据
suozailie =input(' 确认数据在导入的第几列：')
sannian_ = get_line_excel(shuju,suozailie)
#print(sannian)
del sannian_[0]#剔除表头
#print(sannian)
sannian = []
for a in sannian_:#数据标准格式化
    a_ = float(a)
    sannian.append(a_)


#1. 计算三年中后两年的移动求和: 根据数据选取周期，日数据周期为 365，周数据周期为 52，月数据周期为 12
T = int(input(' 数据的时间跨度是？日数据周期为 365，周数据周期为 52，月数据周期为 12：'))
Last_two_years_yidong_Sum = SUM_T_yidong(sannian,T)
print(" 后两年移动求和值：")
print(Last_two_years_yidong_Sum)

#2. 计算后两年按照年为周期的累加数值
#2.1 切割列表
Last_two_years = sannian[T:]

Last_two_years_fenduan_sum = Fenduan_Sum(Last_two_years,T)
#2.2 得到分段累加值列表
print(" 后两年各期分段累加值：")
print(Last_two_years_fenduan_sum)


#3. 生成移动求和的预测值，预测一整年
weilai_one_year = oneline_huigui_only_Y_yuce(Last_two_years_yidong_Sum,T)
print(" 预测未来一年每期的移动求和值：")
print(weilai_one_year)

#4. 根据未来一年的移动求和值反向递减出未来一年每个周期的值
meiyue = []
for a in weilai_one_year:
    meiyue.append(a)
for i in range(1,T):#先减去去年
meiyue[i-1] = meiyue[i-1]-sum(sannian[-T+i:])
#print(meiyue)
for i in list(range(1,T)):#在减去预测年每月累加值
meiyue[i] = meiyue[i]- sum(meiyue[:i])
print(" 预测未来一年每期值：")
print(meiyue)

weilai_one_year_fenduan_sum = Fenduan_Sum(meiyue,T)
print(" 未来一年各期分段累加值：")
print(weilai_one_year_fenduan_sum)

Yidong_two_and_one = [' 两年加预测移动求和项 '] + Last_two_years_yidong_Sum + weilai_one_year
geqileiji_two_and_one = [' 两年加预测各期累加项 '] +Last_two_years_fenduan_sum + weilai_one_year_fenduan_sum
two_and_one =[' 两年加预测各期值 '] + Last_two_years + meiyue
meiyue_yuce = [' 预测未来一年每期值 ']+meiyue

Zong = {}
biao_write_zidian_hang(Zong,Yidong_two_and_one,1)
biao_write_zidian_hang(Zong,geqileiji_two_and_one,2)
biao_write_zidian_hang(Zong,two_and_one,3)
biao_write_zidian_hang(Zong,meiyue_yuce,4)

#for a,b in Zong.items():
    #print(a,b)
YUCE = write_excel(Zong)

print(YUCE)

houzhui = input(" 输入保存格式后缀：")

save_excel(YUCE,lujing,baocun_name,houzhui)
```

### 表格汇总

经常和表格打交道的选手们，应该对表格汇总并不陌生，复制粘贴用的很熟练，但是速度和繁琐程度还是比较麻烦。

这里可以考虑用 python 来解决问题

```
#昨天写的读取一个文件的函数
def read_oneExcel_in(lujing):
    import xlrd
    import re
    WENJIAN = xlrd.open_workbook(lujing)  # 打开文件
YE = WENJIAN.sheets()[0]  # 通过索引顺序获取锁定第一页
    # 获取数据范围（行数和列数）
Hang = YE.nrows  # 获取行数
Lie = YE.ncols  # 获取列数
jian = {}  # 声明存放字典的值也是字典
    # 遍历列表
for hang in range(0, Hang):  # 遍历行数
zhushi_hang = (' 第 ' + str(hang + 1) + ' 行 ')  # 形成行书签
jian[zhushi_hang] = {}  # 每一行作为一个字典
for lie in range(0, Lie):  # 遍历列数
YE_00 = re.split(r"text:'|'", str(YE.cell(hang, lie)))  # 提取表格关键信息
for ccc in YE_00:  # 遍历提取信息形成的过渡列表
if ccc != '':  # 剔除列表里面的无关值
zhushi_lie = (' 第 ' + str(lie + 1) + ' 列 ')  # 形成列书签
if ccc == 'empty:':
                        ccc = ''
if 'number:' in ccc:
                        xxx_long = ccc.index('.')
                        ccc = int(ccc[7:xxx_long])
                    if 'xldate:' in str(ccc):
                        xxx_long_ = str(ccc).index('.')
                        ccc = int(str(ccc)[7:xxx_long_])
                    jian[zhushi_hang][zhushi_lie] = ccc
    return jian
lujing = 'F:\PYTHON\BISU\第四课\数据 '
#geshi = input(' 你要汇总的格式是 xls、xlsx？：')
geshi = 'xlsx'
path = u''+lujing
import os
HOME = []              #我们创建了一个列表，用来装载我们获取到的路径
filelist = os.listdir(path)  # 该文件夹下所有的文件（包括文件夹）
for i in range(0,len(filelist)):  # 遍历所有文件
files = str(filelist[i])
    Olddir = os.path.join(path, files)  # 原来的文件路径
if os.path.isdir(Olddir):  # 如果是文件夹则跳过
continue
    if geshi in files:
        HOME.append(lujing+'\\'+files)   #把路径放到
ALL = []         #我们创建了一个列表用来装载获得的所有表格的数据
#用 for 遍历 HOME 逐一获取路径, 冒号别少了
for i  in HOME:
    #调用昨天写好的读取一个表格的函数
ALL.append(read_oneExcel_in(i))

#创建一个变量装最后要输出的内容
OK = ''
#因为 ALL 是列表，所以我们看信息的时候遍历出来会更加的容易阅读
for i in ALL:
    #因为我们写的读取表格后的数据是字典，所以用字典的遍历方式.items() 会更便于阅读
for a,b in i.items():
        for c,d in b.items():
            OK = OK + str(d) +'\t'
OK = OK + '\n'
print(OK)
input(' 暂停|回车写出 ')

xie = open(lujing+'\\' + 'ADD_ALL.xlsx' , 'w',encoding='utf-8')       # 在目的路径声明一个文件及格式并打开
xie.write(str(OK))                             # 把处理好的信息写入这个文件
xie.close()                                    # 关闭写入操作
```

### 批量录单

之前有个做贸易的朋友问我，有没有办法批量录单。

他每天都要花上几个钟，来填一些固定的表单：供应商、联系方式、物流编号之类的……

每天同样的重复性工作，除了耗时间，效率低下之外，还会有复制错的情况存在。

长期以往之下，视力从 5.0 到 3.0 不是梦想。

当然，我觉得简单重复的事情交给机器做是最靠谱的。

第一是效率快，几分钟不到完成你几个钟的工作。

第二是准确性高，除非有 BUG，否则基本上不存在看错、误操作之类的低级错误。

```
import os                                                                                #导入 os 模块
import os.path                                                                           #导入 os.path 模块
import xlrd                                                                              #导入第三方库 xlrd
import re
import sys                                                                               #导入 sys 模块
import glob                                                                              #导入 glob 模块
#读取路径内同格式 EXCEL 为四级字典文件名（表名（行数（列数（值））））返回字典
def read_lujing_in(wenjian_lujing,houzhuichangdu,houzhuiming):                            #创建一个获取文件夹内 xls 格式文件路径的函数
houzhuichangdu = int(houzhuichangdu)
    houzhuiming = str('.'+ houzhuiming)
    wenjian_name = []                                                                    #创建一个空列表用来装载获取的路径
    for lujing,mingcheng,wenjianmings in os.walk(wenjian_lujing):                        #从 os 获取的信息中遍历路径，名称，文件名
        # 把指定后缀长度为指定后缀名的文件名找出来赋给文件名
        wenjianmings = filter(lambda wenjianming:wenjianming[-(houzhuichangdu+1):] == houzhuiming,wenjianmings)
        wenjianmings = map(lambda wenjianming:os.path.join(lujing,wenjianming),wenjianmings)#把文件名拼接到文件夹路径形成新的文件名
        wenjian_name.extend(wenjianmings)                                                #把生成的文件名放进路径列表
    a001 = wenjian_lujing                                                                #用 a001 传递路径
    a002 = len(a001)+1                                                                   #计算路径的字符长度
    b001 = houzhuichangdu                                                                #用 b001 传递后缀长度
    b002 = int(b001) + 1                                                                 #真实后缀包括. 占一位
    cunfang = {}                                                                         #用一个字典存放数据
    for a_i in range(0, len(wenjian_name)):                                              #遍历由文件数组成的数组
        jian = wenjian_name[a_i][len(a001) + 1:-b002] + '_new'                           #编辑字典的键为新文件名
        WENJIAN = xlrd.open_workbook(wenjian_name[a_i])                                  # 打开文件
        YE = WENJIAN.sheets()[0]                                                         # 通过索引顺序获取锁定第一页
        # 获取数据范围（行数和列数）
        Hang = YE.nrows                                                                  # 获取行数
        Lie = YE.ncols                                                                   # 获取列数
        cunfang[jian] = {}                                                               #声明存放字典的值也是字典
        # 遍历列表
        for hang in range(0, Hang):                                                      # 遍历行数
            zhushi_hang = (' 第 ' + str(hang + 1) + ' 行 ')                      # 形成行书签
            cunfang[jian][zhushi_hang] = {}                                              # 每一行作为一个字典
            for lie in range(0, Lie):                                                    # 遍历列数
                YE_00 = re.split(r"text:'|'", str(YE.cell(hang, lie)))                   # 提取表格关键信息
                for ccc in YE_00:                                                        # 遍历提取信息形成的过渡列表
                    if ccc != '':                                                        # 剔除列表里面的无关值
                       zhushi_lie = (' 第 ' + str(lie + 1) + ' 列 ')            # 形成列书签
                       if ccc == 'empty:':
                                ccc = ''
                       if 'number:' in ccc:
                                xxx_long = ccc.index('.')
                                ccc = int(ccc[7:xxx_long])
                            cunfang[jian][zhushi_hang][zhushi_lie] = ccc
return cunfang        #返回这个字典



#改变指定字典内的指定表格字典的指定行列的最终值 执行操作不返回
def change(cunfang_name,wenjian_name,hang,lie,change_zhi):                               #创建一个修改指定行列为特定值的函数
    hang = (' 第 ' + str(hang) + ' 行 ')                                                     #创建按第几行为行注释的索引
    lie = (' 第 ' + str(lie) + ' 列 ')                                                       #创建按第几列为列注释的索引
    change_zhi = str(change_zhi)                                                         #指定修改值
    cunfang_name[wenjian_name][hang][lie] = change_zhi                                   #把特定文件下的表的某行某列换为某值
#把表格字典转化成表格样式的一整个字符串 返回字符串
def write_excel(A):                                                                      #创建一个转换字典为特定字符表的函数
    for B,C in A.items():                                                                #遍历表字典内的行：列键值对
        Hang = int(len(A))                                                               #获得行数
        Lie  = int(len(C))                                                               #获得列数
        Lie_Biao = []                                                                    #创建中转列：值对的列表
        Zuizhong = []                                                                    #创建最终所有列值对字符串列表
        for hhh in A.values():                                                           #遍历表中的列字典
            for lll in hhh.values():                                                     #遍历列字典的值
                Lie_Biao.append(str(lll))                                                #把值字符串化添加到值列表
        for hangshu in range(0, Hang + 1):                                               #定义行数为跳转步长
            guodu = Lie_Biao[hangshu * Lie:hangshu * Lie + Lie]                          #定义按每行转化成过渡切片
            jiange_1 = '\t'                                                              #定义一个制表符
            meiyihang = jiange_1.join(guodu)                                             #用制表符来连接切片中的每一列
            Zuizhong.append(meiyihang)                                                   #把由每一列连起来的每一行添加到最终列表
        jiange_2 = '\n'                                                                  #定义一个换行符
        zhenggebiao = jiange_2.join(Zuizhong)                                            #用换行符来连接最终列表内每个行元素
    return zhenggebiao                                                               #返回一个表示整个表的字符串
#把特定的字符串写入特定路径按特定名和格式保存
def save_excel(baocunduixiang,wenjianjialujing,baocunming,geshihouzhui):
    baocunming = str(baocunming)                                                         #指定保存名
    LUJING_NEW = wenjianjialujing + '\\' + baocunming + '.'+ geshihouzhui                #拼装最终路径
    f = open(LUJING_NEW, 'w')                                                            # 在目的路径声明一个文件及格式并打开
    f.write(baocunduixiang)                                                              # 把处理好的信息写入这个文件
    f.close()                                                                            # 关闭写入操作
print(" 导入需要批处理的表格 ")
a001 = input(" 文件夹路径：")
c001 = input(" 所查找文件的格式后缀名：")
b001 = len(c001)

print(" 数据导入成功：")
A = read_lujing_in(a001,b001,c001)
for aa,bb in A.items():
    print(aa)
    print(write_excel(bb))

xuanze_1 = input(" 是否继续程序（no or 任意继续）:")
if xuanze_1 == 'no':
    print('- - - END - - -')
else:
    print(" 导入修改模板 "+"\n"+"(模版修改的坐标范围要在被修改表的坐标范围内)")
    a002 = input(" 文件夹路径：")
    c002 = input(" 所查找文件的格式后缀名：")
    b002 = len(c002)
    B = read_lujing_in(a002, b002, c002)  # 得到三元镶套的数据字典
for cc in B.values():
        print(" 导入修改模板成功：")
        print(len(cc))
        #显示修改模板以验证并遍历 A 进行修改
for c_i in range(2, len(cc)+1):
            LH = ' 第 ' + str(c_i) + ' 行 '
L1 = str(cc[LH][' 第 1 列 '])
            L2 = str(cc[LH][' 第 2 列 '])
            L3 = str(cc[LH][' 第 3 列 '])
            L4 = str(cc[LH][' 第 4 列 '])
            print(LH, L1, L2, L3, L4)
            # 遍历 A 修改指定值
for aa_i in A.keys():
                if L1 in aa_i:
                    change(A, aa_i,L2,L3,L4)
    print(" 数据修改成功：")
    for aa, bb in A.items():
        print(aa)
        print(write_excel(bb))
    xuanze_2 = input(" 是否导出数据（ no or 任意继续）:")
    if xuanze_1 == 'no':
        print('- - - END - - -')
    else:
        LUJING = a002
        geshi = 'xlsx'
#把 A 内每个表都转化成列表字符串并输出文件
for BB, CC in A.items():
            zhenggebiao = write_excel(CC)
            #print(zhenggebiao)
save_excel(zhenggebiao,LUJING,BB,geshi)

input(" 导出成功|回车结束 ")
```