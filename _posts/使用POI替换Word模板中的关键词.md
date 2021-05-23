---
title: 使用POI替换Word模板中的关键词
date: 2020-07-10 22:23:34
tags: [Java, Word]
---

这里的POI不是Point Of Interest兴趣点，而是针对Microsoft Documents的开源Java API，当前最新版本是4.1.2，官网[指路]( http://poi.apache.org/ )。Word模板格式是`.docx`，所以重点讨论POI中的XWPFDocument。JDK版本是JDK 14。IDE使用IntelliJ IDEA 2020.1.2。<!--more-->

## 从数据库提取信息

首先，连接Oracle数据库，查看待操作表的字段，拟定sql语句在查询窗口运行查看结果。然后，导入ojdbc10.jar和fastjson*-1.2.7.*jar，编写代码连接数据库并执行查询语句，组合每一个JSONObject，构造JSONArray返回。这个JSONArray中的键值对用来替换Word模板中的关键词。

```java
// 连接数据库，查询的代码
package ManipulateDB;

import com.alibaba.fastjson.JSONArray;
import com.alibaba.fastjson.JSONObject;
import java.sql.*;

public class OracleDB {
    private String url = "jdbc:oracle:thin:@ip地址:端口号:SID";
    private String username = "your_name";
    private String password = "your_password";
    
    /**
     * 连接oracle数据库
     * @return 返回Connection实例
     */
    public Connection getConnect(){
        Connection connect = null;
        try{
            //建立驱动
            String driverName = "oracle.jdbc.driver.OracleDriver";
            Class.forName(driverName);
            //连接
            connect = DriverManager.getConnection(url, username, password);
        }catch (Exception e){
            e.printStackTrace();
        }
        return connect;
    }

    /**
     * 根据TBYBH编号查询DLTB表获取信息：编号，坐落，面积，地类编码，地类名称
     * @param idStr
     * @return 返回一个JSONArray，包含上述信息
     */
    public JSONArray query(String idStr) throws Exception{
        JSONArray infos = new JSONArray();
        try{
            Connection connect = getConnect();
            Statement stmt = connect.createStatement();
            String sql = "select TBYBH,ZLDWDM,ZLDWMC,SHAPE_AREA_1,SHDLBM,SHDLMC from DLTB where TBYBH= '"+idStr+"'";
            ResultSet rs = stmt.executeQuery(sql);

            while(rs.next()){
                String TBYBH = rs.getString("TBYBH");
                String ZLDWDM = rs.getString("ZLDWDM").substring(0,12);//截取前12位
                int SHAPE_AREA_1 = (int)Math.floor(rs.getInt("SHAPE_AREA_1"));//保留整数位
                String SHDLBM = rs.getString("SHDLBM");
                String SHDLMC = rs.getString("SHDLMC");
                String country = rs.getString("ZLDWMC");

                String address = this.queryAddress(ZLDWDM.substring(0,6));
                address += this.queryAddress(ZLDWDM.substring(0,9));
                address += country;

                JSONObject info = new JSONObject();
                info.put("TBYBH",TBYBH);
                info.put("ZLDWDM",ZLDWDM);
                info.put("SHAPE_AREA_1",SHAPE_AREA_1);
                info.put("SHDLBM",SHDLBM);
                info.put("SHDLMC",SHDLMC);
                info.put("ADDRESS",address);

                infos.add(info);
            }

            // 数据库查询后一定要关闭资源，不然数据库会限制连接数目，从而中断程序
            connect.close();
            stmt.close();
            rs.close();
        }catch (SQLException e){
            e.printStackTrace();
        }
        return infos;
    }

    public static void main(String[] args) throws Exception{
        OracleDB orcltool = new OracleDB();
        JSONArray infos = orcltool.query("310117120F04318");
        System.out.println(infos);
    }
```
## 替换模板中的关键词

POI结构说明如下，更多信息查看Components，地址[指路](http://poi.apache.org/components/index.html)。

| 名称 |          说明           |
| :--: | :---------------------: |
| HSSF | 读写Excel .xls格式文档  |
| XSSF | 读写Excel .xlsx格式文档 |
| HWPF |  读写Word.doc格式文档   |
| XWPF |  读写Word.docx格式文档  |
| HSLF |   读写PPT.ppt格式文档   |
| XLSF |  读写PPT.pptx格式文档   |

本文的模板文档格式是`.docx`，主体是一个表格，于是遍历每一个单元格，若匹配到关键词，则删除原有文本，填入自定义文本。没错，XWPFDocument没有提供replace之类的api，只能删除原有文本再填入新文本。`.docx`文档实际上是一个压缩包。若加载多媒体资源，比如图片时，如果没有展示出来，可以将后缀名改为.zip再解压便可以看到内部结构，查看多媒体资源是否写入成功。

```java
   /**
     * 替换对应信息
     * @param docx
     * @param info
     * @return
     */
	public void replaceInTable(XWPFDocument docx,JSONObject info) throws Exception{
        List<XWPFTable> tables = docx.getTables();
        XWPFTable table = tables.get(tables.size()-1);
        List<XWPFTableRow> rows;
        List<XWPFTableCell> cells;

        rows = table.getRows();
        for (XWPFTableRow row : rows) {
            cells = row.getTableCells();
            for (XWPFTableCell cell : cells) {
                String text = cell.getText();
                if(text.equals("TBYBH")){
                    cell.removeParagraph(0);
                    cell.setText(info.getString("TBYBH"));
                    System.out.println("TBYBH修改成功:  "+info.getString("TBYBH"));
                }else if(text.equals("ZLDWDM")){
                    cell.removeParagraph(0);
                    cell.setText(info.getString("ADDRESS"));
                }else if(text.equals("SHAPE_AREA_1")){
                    cell.removeParagraph(0);
                    cell.setText(info.getString("SHAPE_AREA_1"));
                }else if(text.equals("SHDLBM")){
                    cell.removeParagraph(0);
                    cell.setText(info.getString("SHDLBM"));
                }else if (text.equals("SHDLMC")){
                    cell.removeParagraph(0);
                    cell.setText(info.getString("SHDLMC"));
                }else if(text.equals("picture")){
                    this.addPics(cell,info.getString("TBYBH"));
                }
            }
        }
    }

   /**
     * 删除cell原内容，添加图片
     * @param cell
     * @param TBYBH
     * @throws Exception
     */
    public void addPics(XWPFTableCell cell,String TBYBH)throws Exception{
        cell.removeParagraph(0);
        XWPFParagraph paragraph = cell.addParagraph();
        XWPFRun run = paragraph.createRun();
        run.setText("现场照片：");
        run.addBreak();

        String picspath = this.dirpath+"\\"+TBYBH;
        String[] pics = new File(picspath).list();
        for(int i=0;i<4 && i<pics.length;i++){
            InputStream is = new FileInputStream(picspath+"\\"+pics[i]);
            if(i == 0){
                run.setText("图一：");
            }else if(i == 1){
                run.setText("图二：");
            }else if(i == 2){
                run.addBreak();
                run.setText("图三：");
            }else if(i == 3){
                run.setText("图四：");
            }
            run.addPicture(is,XWPFDocument.PICTURE_TYPE_JPEG,pics[i], Units.toEMU(140),Units.toEMU(140));
            run.setText(" ");
            is.close();
        }
    }
```

## 合并文档

一个一个word文档打印起来要点击很多次，于是选择合并为一个文档。选择使用Python的win32模块，注意导入相关依赖。待合并的word文档统一放进一个文件夹，再指定目的文档地址即可。

```python
# author：EduTech
# link：https://zhuanlan.zhihu.com/p/100588511

# 依赖：
# pip install pywin32
# pip install pypiwin32

import win32com.client as win32
import os
word = win32.gencache.EnsureDispatch('Word.Application')
#启动word对象应用
word.Visible = False
path = r'D:\software\doc'
files = []
for filename in os.listdir(path):
    filename = os.path.join(path,filename)
    files.append(filename)
#新建合并后的文档
output = word.Documents.Add()
for file in files:
    output.Application.Selection.InsertFile(file)#拼接文档
#获取合并后文档的内容
doc = output.Range(output.Content.Start, output.Content.End)
output.SaveAs('D://software//doc//result.docx') #保存
output.Close()
```

## 结束语

整个项目从下载IDEA开始，中间经历了数据库用户名称密码错误，破解IDEA，熟悉IDEA，下载JDK14，POI依赖包的查找，doc转docx，查阅POI官方文档，查找前人博客等一系列问题，直到生成最终的文档，总共花费了三天时间。其中有一天半的时间都卡在怎么样复制旧表格添加到文档末尾，全英文的官方文档看的我头大，越看越急躁，越静不下心来。回想起花了两三个小时看完pyshp优美的官方文档，讲的细致而且举例丰富，IDLE直接可以上手尝试，哪怕是全英文也理解的非常快。下次我学习官方文档的时候，就开着IDEA建一个项目试它的API，应该效果会好很多。

IDEA的界面做的比MyEclipse好看多了。

Word文档的结构好复杂，写一份好看的word文档要花好多时间，希望将来能流行用简单又美观的Markdown文档。

欢迎联系我347335189@qq.com。

看得开心!