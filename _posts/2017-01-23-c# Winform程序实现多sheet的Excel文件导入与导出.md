---
layout:     post
title:      "c# Winform程序实现多sheet的Excel文件导入与导出"
date:       2017-01-23 12:00:00
author:     "邵正"
header-img: "img/home-bg-o.jpg"
tags:
    - C#
    - mysql
---

| 主题     | 概要                                                                                   |
| -------- | -------------------------------------------------------------------------------------- |
| C#       | excel导入到mysql，mysql导出到excel                                                     |
| -------- | ---                                                                                    |
| **编辑** | **时间**                                                                               |
| 新建     | 20170123                                                                               |
| -------- | ---                                                                                    |
| **序号** | **参考资料**                                                                           |
| 1        | http://download.csdn.net/detail/nanzhaonan/5403457（左侧导航菜单）                     |
| 2        | https://msdn.microsoft.com/zh-cn/library/system.data.oledb(v=vs.110).aspx (OleDB MSDN) |
| 3        | https://www.connectionstrings.com/（连接字符串）                                       |
| 4        | www.newtonsoft.com/json （json.net）                                                   |


年前公司没什么事儿，自告奋勇用C#来实现一个客户端工具，主要设想是建立个模型库，实现excel文件的导入导出，用来自动化管理（相对手工来说）各局点的配置数据。
好久没有写C#程序了，所以这也算从头开始了，下面把实现过程中，碰到的感觉有点儿价值的东西记录下来，做个总结，也方便以后备查。


## Excel导入##
第一次接触OLEDB这个东西，它的全称是Object Linking and Embedding Database，对象链接和嵌入数据库。理解得也不深，简单来说是微软提供的一种能够针对所有类型数据进行读写操作的一种渠道，能够以SQL语句访问非SQL数据类型。
把一个excel导入到Mysql的过程是，先通过OleDB获取这个excel的sheet名字，返回一个列表，然后再按这个名字列表依次去读每个sheet对应的具体内容到DataSet中，再把DataDet中的数据插到数据库中。

下面把这一过程分成两步进行说明：

### Excel到DataSet ###
读sheet名字的过程：

```
  public ArrayList getExcelSheetNames(string filePath)
        {
            ArrayList arrayNames = new ArrayList();           
              string strConn = getExcelOleDBConnectStr(filePath);
            DataTable tb=null;

            try
            {
                OleDbConnection conn = new OleDbConnection(strConn);
                conn.Open();
                tb = conn.GetOleDbSchemaTable(OleDbSchemaGuid.Tables, null);
                foreach (System.Data.DataRow drow in tb.Rows)
                {
                    string sheetName = drow["TABLE_NAME"].ToString().Trim();
                    int pos = sheetName.LastIndexOf('$');
                    if (pos!=-1)
                    {
                        arrayNames.Add(sheetName.Substring(0, pos));                       
                    }                    
                }
            }
            catch (System.Exception ex)
            {
                MessageBox.Show(ex.Message);
            }

            return arrayNames;
          
        }


```

比如我有一个excel包含了两个sheet:
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcwMTIzMTQyMjE2Mjcx)
通过上面的函数就能获得这两个sheet的名字；

通过这种方式，获取的sheetName会有些莫名其妙的字符串，
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcwMTIzMTQyMzE2MTQy)


而真正的sheetName是这样的：
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcwMTIzMTQyMzQyMTQ3)

我依赖于sheetName名字后有一个”$”字符进行过滤，至于真正的会出现多余sheetName的原因就不求甚解了。

这是个连接excel数据源的字符串，网上转来转去的也没个标准说明。

```
private string getExcelOleDBConnectStr(string filePath)
        {
            string strConn = "Provider=Microsoft.ACE.OLEDB.12.0;"
               + "Data Source=" + @filePath + ";" + "Extended Properties='Excel 12.0; HDR=Yes; IMEX=1'";

            return strConn;
        }

```
发现了一个很好用的网址：https://www.connectionstrings.com/
里面包含了开发者要用到的所有连接字符串，选择excel进去就会有各种说明。

获得sheetName后，就是把sheet表里面的数据，挨个读到DataSet中。

```
public DataSet excelToDataSet(string filePath,string tobeOpenSheet)
        {
            string strConn = getExcelOleDBConnectStr(filePath);
            DataSet ds = null;
            OleDbConnection conn = new OleDbConnection(strConn);

            try
            {              
                conn.Open();
                string strExcel = "";
                OleDbDataAdapter myCommand = null;
                strExcel = "select * from [" + tobeOpenSheet + "$]";
                myCommand = new OleDbDataAdapter(strExcel, strConn);
                ds = new DataSet();

                myCommand.Fill(ds);
            }
            catch (System.Exception ex)
            {
                MessageBox.Show(ex.Message);
            }
            finally
            {
                conn.Close();
            }

            return ds;
        }


```
### DataSet到Mysql ###

在使用C#连接Mysql之前，要先引用Mysql 的ADO.NET驱动程序：
可以在：https://www.mysql.com/选择下载。

然后添加引用：
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcwMTIzMTQyNTU1NDk0)

假设我用的数据库是mysql，mysql中有这么一个表，含有7个字段：
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcwMTIzMTQyNjI3NDc5)

则我给这个表建立一个对应的Bean（Java中的概念，C#中好像没有）

```
class MQModbusTableBean:MQBeanBase
    {
        private int sn;
        private string modbusAddress;
        private int nb;
        private string opertionType;
        private int paramTag;
        private int crclh;
        private string snmpOid;

        public string SnmpOid
        {
            get { return snmpOid; }
            set { snmpOid = value; }
        }

        public int Crclh
        {
            get { return crclh; }
            set { crclh = value; }
        }

        public int ParamTag
        {
            get { return paramTag; }
            set { paramTag = value; }
        }

        public string OpertionType
        {
            get { return opertionType; }
            set { opertionType = value; }
        }

        public int Nb
        {
            get { return nb; }
            set { nb = value; }
        }
        public string ModbusAddress
        {
            get { return modbusAddress; }
            set { modbusAddress = value; }
        }

        public int Sn
        {
            get { return sn; }
            set { sn = value; }
        }

        public MQModbusTableBean()
        {
            modbusAddress = "";
            opertionType = "";
            snmpOid = "";
        }

        public override string toInsertSql()
        {
            string sql = string.Format("insert into {0} (SN,MODBUSADDRESS,NB,OPERATION_TYPE,PARAMTAG,CRCLH,SNMPOID) values"
                + "({1},'{2}',{3},'{4}',{5},{6},'{7}');\n",
                MQCommonConst.COLLECTOR_MODBUS_TABLE, sn, modbusAddress, nb, opertionType, paramTag, crclh,snmpOid);
            return sql;
        }
    }


```

这里定义了一个Bean基类，因为我有很多这样的表，成员函数只有一个简单的一个插入操作：

```
abstract class MQBeanBase
    {
        public abstract string toInsertSql();        
    }

```
用一个静态成员函数，根据DataSet数据来填充这个表的Bean：

```
  public static ArrayList fillModbusTable(DataSet dataSet)
        {
            ArrayList modbusTableList = new ArrayList();

            foreach (DataTable table in dataSet.Tables)
            {
                removeEmptyRows(table);
                foreach (DataRow row in table.Rows)
                {
                    MQModbusTableBean modbusBean = new MQModbusTableBean();                 

                    foreach (DataColumn column in table.Columns)
                    {
                        string columnName = column.ColumnName;
                        string columnDisplay = row[column].ToString().Trim();

                        if (columnDisplay.Equals(""))
                        {
                            continue;
                        }

                        switch (columnName)
                        {
                            case "SN":
                                {
                                    modbusBean.Sn = int.Parse(columnDisplay);
                                    break;
                                }
                            case "MODBUSADDRESS":
                                {
                                    modbusBean.ModbusAddress = columnDisplay;
                                    break;
                                }
                            case "NB":
                                {
                                    modbusBean.Nb = int.Parse(columnDisplay);
                                    break;
                                }
                            case "OPERATION_TYPE":
                                {
                                    modbusBean.OpertionType = columnDisplay;
                                    break;
                                }                           
                            case "CRCLH":
                                {
                                    modbusBean.Crclh = int.Parse(columnDisplay);
                                    break;
                                }
                            case "SNMPOID":
                                {
                                    modbusBean.SnmpOid =columnDisplay;
                                    break;
                                }
                            default:
                                break;
                        }
                    }
                    modbusTableList.Add(modbusBean);
                }
            }

            return modbusTableList;
        }


```
这里把excel中的每一行内容放到一个bean中，最后返回所有的bean列表。

对于一个sheet表，可能会出现空行，需要移除空行：

```
private static void removeEmptyRows(DataTable dt)
        {
            List<DataRow> removelist = new List<DataRow>();
            for (int i = 0; i < dt.Rows.Count; i++)
            {
                bool isNull = true;
                for (int j = 0; j < dt.Columns.Count; j++)
                {
                    if (!string.IsNullOrEmpty(dt.Rows[i][j].ToString().Trim()))
                    {
                        isNull = false;
                    }
                }
                if (isNull)
                {
                    removelist.Add(dt.Rows[i]);
                }
            }
            for (int i = 0; i < removelist.Count; i++)
            {
                dt.Rows.Remove(removelist[i]);
            }
        }


```
最后一步是对bean列表入库操作：

```
public void insertEachTableNewRecord(ArrayList beanList)
        {
            StringBuilder sqlIntergate = new StringBuilder();

            foreach(MQBeanBase bean in beanList)
            {
                sqlIntergate.Append(bean.toInsertSql());
            }

            if (!sqlIntergate.ToString().Equals(""))
            {
                executeNonQueryCommd(sqlIntergate.ToString());
            }
        }


```
传一个beanList参数，把所有bean的插入sql合在一起，执行数据库的非查询命令。

与Mysql打交道的主要函数：
获取连接到Mysql数据库的连接字符串：

```
protected MySqlConnection getMysqlConn()  //--连接数据库
        {           
            if (sqlConnStr.Equals(""))
            {
               sqlConnStr=MQModifyAddressConfig.parseAddressInfo().toConnectDBStr();
            }
            
            MySqlConnection myConn = new MySqlConnection(sqlConnStr);
            return myConn;
        }

```
连接数据库的地址存在一个json文件中，可以手动配置，也可以在界面上修改，后面会说解析json的方式。

执行SQL命令：

```
public void executeNonQueryCommd(string sqlCommd)         //--执行sql命令
        {
           MySqlConnection mySqlConn=null;
           MySqlCommand mySqlCommd=null;
            try
            {
                mySqlConn = this.getMysqlConn();
                mySqlConn.Open();             
                mySqlCommd = new MySqlCommand(sqlCommd, mySqlConn);
                mySqlCommd.ExecuteNonQuery();
            }
            catch (System.Exception ex)
            {
                MessageBox.Show(ex.Message);
            }
            finally
            {
              mySqlCommd.Dispose();
              mySqlConn.Close();
              mySqlConn.Dispose();
            }  
        }


```

查询sql，获取一个DataSet：

```
public DataSet queryCommand(string sqlCommd)
        {
            DataSet ds = new DataSet();
            MySqlDataAdapter mySqlAdapter = null;
            MySqlConnection mySqlConn=null;       

            try
            {
                mySqlConn = this.getMysqlConn();
                mySqlConn.Open();               
                mySqlAdapter = new MySqlDataAdapter(sqlCommd, mySqlConn);
                mySqlAdapter.Fill(ds, "table"); 
            }
            catch (System.Exception ex)
            {
                MessageBox.Show(ex.Message);
            }
            finally
            {
                mySqlConn.Close();
            }

            return ds;
        }


```

只返回一个表：

```
public DataTable queryTableBySql(string sqlCommd)
        {
            DataTable tb = new DataTable();

            if (sqlCommd.Equals(""))
            {
                return tb;
            }

            DataSet ds = queryCommand(sqlCommd);

            if (ds.Tables.Count > 0)
            {
                tb = ds.Tables[0];
            }

            return tb;
        }


```
下面是合在一起的，所有与excel与数据库打交道的基类：

```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Data;
using System.Windows.Forms;
using MySql.Data.MySqlClient;
using System.Data.OleDb;
using System.Collections;

namespace MQHelper
{
    class ModelQuery
    {
        private static string sqlConnStr ="";
       

        protected MySqlConnection getMysqlConn()  
        {           
            if (sqlConnStr.Equals(""))
            {
               sqlConnStr=MQModifyAddressConfig.parseAddressInfo().toConnectDBStr();
            }
            
            MySqlConnection myConn = new MySqlConnection(sqlConnStr);
            return myConn;
        }

        public static void changedSqlConnStr(string sqlStr)
        {
            sqlConnStr = sqlStr;
        }

        public DataSet queryCommand(string sqlCommd)
        {
            DataSet ds = new DataSet();
            MySqlDataAdapter mySqlAdapter = null;
            MySqlConnection mySqlConn=null;       

            try
            {
                mySqlConn = this.getMysqlConn();
                mySqlConn.Open();               
                mySqlAdapter = new MySqlDataAdapter(sqlCommd, mySqlConn);
                mySqlAdapter.Fill(ds, "table"); 
            }
            catch (System.Exception ex)
            {
                MessageBox.Show(ex.Message);
            }
            finally
            {
                mySqlConn.Close();
            }

            return ds;
        }

        public void executeNonQueryCommd(string sqlCommd)         
        {
           MySqlConnection mySqlConn=null;
           MySqlCommand mySqlCommd=null;
            try
            {
                mySqlConn = this.getMysqlConn();
                mySqlConn.Open();             
                mySqlCommd = new MySqlCommand(sqlCommd, mySqlConn);
                mySqlCommd.ExecuteNonQuery();
            }
            catch (System.Exception ex)
            {
                MessageBox.Show(ex.Message);
            }
            finally
            {
              mySqlCommd.Dispose();
              mySqlConn.Close();
              mySqlConn.Dispose();
            }  
        }

        public DataTable queryTableBySql(string sqlCommd)
        {
            DataTable tb = new DataTable();

            if (sqlCommd.Equals(""))
            {
                return tb;
            }

            DataSet ds = queryCommand(sqlCommd);

            if (ds.Tables.Count > 0)
            {
                tb = ds.Tables[0];
            }

            return tb;
        }


        private string getExcelOleDBConnectStr(string filePath)
        {
            string strConn = "Provider=Microsoft.ACE.OLEDB.12.0;"
               + "Data Source=" + @filePath + ";" + "Extended Properties='Excel 12.0; HDR=Yes; IMEX=1'";

            return strConn;
        }

        public ArrayList getExcelSheetNames(string filePath)
        {
            ArrayList arrayNames = new ArrayList();
            string strConn = getExcelOleDBConnectStr(filePath);
            DataTable tb=null;

            try
            {
                OleDbConnection conn = new OleDbConnection(strConn);
                conn.Open();
                tb = conn.GetOleDbSchemaTable(OleDbSchemaGuid.Tables, null);
                foreach (System.Data.DataRow drow in tb.Rows)
                {
                    string sheetName = drow["TABLE_NAME"].ToString().Trim();
                    int pos = sheetName.LastIndexOf('$');
                    if (pos!=-1)
                    {
                        arrayNames.Add(sheetName.Substring(0, pos));                       
                    }                    
                }
            }
            catch (System.Exception ex)
            {
                MessageBox.Show(ex.Message);
            }

            return arrayNames;
          
        }


        public DataSet excelToDataSet(string filePath,string tobeOpenSheet)
        {
            string strConn = getExcelOleDBConnectStr(filePath);
            DataSet ds = null;
            OleDbConnection conn = new OleDbConnection(strConn);

            try
            {              
                conn.Open();
                string strExcel = "";
                OleDbDataAdapter myCommand = null;
                strExcel = "select * from [" + tobeOpenSheet + "$]";
                myCommand = new OleDbDataAdapter(strExcel, strConn);
                ds = new DataSet();

                myCommand.Fill(ds);
            }
            catch (System.Exception ex)
            {
                MessageBox.Show(ex.Message);
            }
            finally
            {
                conn.Close();
            }

            return ds;
        }
    }
}

```


## Excel导出 ##

从Mysql到DataSet的过程不提，下面是如何从DataSet导出到Excel。

### DataSet到Excel ###

```
public static void exportExcel(System.Data.DataSet ds, string excelFileFullPath, ArrayList sheetsNameList,Hashtable sheetsBeSave)
        {
            if (string.IsNullOrEmpty(excelFileFullPath))
            {
                return ;
            }

            Microsoft.Office.Interop.Excel.Application excelAp=null;

            try
            {               
                excelAp= getExcelApplication();
                Workbook excelBook = excelAp.Workbooks.Add(true);

                //--倒序，为了确保excel中sheet按正序排列
                for (int i = sheetsNameList.Count-1; i >=0; i--)
                {
                    string sheetName = sheetsNameList[i] as string;
                    System.Data.DataTable tb = ds.Tables[sheetName];
                    Worksheet excelSheet = excelBook.Sheets.Add(Missing.Value, Missing.Value, Missing.Value, Missing.Value);
                    excelSheet.Name = sheetName;

                    int colIndex = 0;

                    ArrayList colNames = sheetsBeSave[sheetName] as ArrayList;
                    foreach (string colName in colNames)
                    {
                        colIndex++;
                        excelSheet.Cells[1, colIndex] = colName;
                    }

                    Microsoft.Office.Interop.Excel.Range titleRange = excelSheet.Range[excelSheet.Cells[1, 1], excelSheet.Cells[1, colNames.Count]];
                    titleRange.Interior.Color = Color.FromArgb(204, 232, 207); //设置豆沙绿颜色
                    Microsoft.Office.Interop.Excel.Range sheetRange = null;

                    int rowNumber = tb.Rows.Count; //不包括字段名 
                    int columnNumber = tb.Columns.Count;

                    //--表中有数据才处理
                    if (rowNumber>0 && columnNumber>0)
                    {
                        object[,] objData = new object[rowNumber, columnNumber];
                        for (int r = 0; r < rowNumber; r++)
                        {   
                            for (int c = 0; c < columnNumber; c++)
                            {
                                objData[r, c] = tb.Rows[r][c].ToString().Trim();
                            }
                        }

                        sheetRange = excelSheet.Range[excelSheet.Cells[2, 1], excelSheet.Cells[rowNumber + 1, columnNumber]];
                        sheetRange.Value2 = objData;

                    }
                }

                excelBook.SaveAs(excelFileFullPath, Missing.Value, Missing.Value, Missing.Value, Missing.Value,
                   Missing.Value, XlSaveAsAccessMode.xlNoChange, Missing.Value, Missing.Value, Missing.Value,
                   Missing.Value, Missing.Value);

                excelBook.Close();

            }
            catch (System.Exception ex)
            {
                MessageBox.Show(ex.ToString());
            	
            }
            finally
            {
                closeExcel(excelAp);
            }

        }


```

这个函数的每一个参数是DataSet数据，第二个参数是保存的路径，注意这两个参数：
ArrayList sheetsNameList 指的是要保存多少个sheet。
Hashtable sheetsBeSave 是一个map，包含每一个sheet对应的字段，也即表头的名字。

完整的导出excel的类：

```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Data;
using System.Reflection;
using Microsoft.Office.Interop.Excel;
using System.Collections;
using System.IO;
using System.Windows.Forms;
using System.Runtime.InteropServices; //--DllImport
using System.Drawing;

namespace MQHelper
{
    class MQExcelOperation
    {
        [DllImport("user32.dll", CharSet = CharSet.Auto,SetLastError = true)]
        static extern int GetWindowThreadProcessId(IntPtr hWnd, out int lpdwProcessId);


       
        public static Microsoft.Office.Interop.Excel.Application getExcelApplication()
        {
            Microsoft.Office.Interop.Excel.Application excelAp = new Microsoft.Office.Interop.Excel.Application();
            excelAp.Visible = false;
            //excelAp.DisplayAlerts = true;
            //excelAp.UserControl = true;
            return excelAp;
        }

        
        public static void closeExcel(Microsoft.Office.Interop.Excel.Application excelAp)
        {
            if (excelAp != null)
            {              
                excelAp.Quit();
                int lpdwProcessId;
                GetWindowThreadProcessId(new IntPtr(excelAp.Hwnd), out lpdwProcessId);
                System.Diagnostics.Process.GetProcessById(lpdwProcessId).Kill();
            }
        }
   
        public static void exportExcel(System.Data.DataSet ds, string excelFileFullPath, ArrayList sheetsNameList,Hashtable sheetsBeSave)
        {
            if (string.IsNullOrEmpty(excelFileFullPath))
            {
                return ;
            }

            Microsoft.Office.Interop.Excel.Application excelAp=null;

            try
            {               
                excelAp= getExcelApplication();
                Workbook excelBook = excelAp.Workbooks.Add(true);

               
                for (int i = sheetsNameList.Count-1; i >=0; i--)
                {
                    string sheetName = sheetsNameList[i] as string;
                    System.Data.DataTable tb = ds.Tables[sheetName];
                    Worksheet excelSheet = excelBook.Sheets.Add(Missing.Value, Missing.Value, Missing.Value, Missing.Value);
                    excelSheet.Name = sheetName;

                    int colIndex = 0;

                    ArrayList colNames = sheetsBeSave[sheetName] as ArrayList;
                    foreach (string colName in colNames)
                    {
                        colIndex++;
                        excelSheet.Cells[1, colIndex] = colName;
                    }

                    Microsoft.Office.Interop.Excel.Range titleRange = excelSheet.Range[excelSheet.Cells[1, 1], excelSheet.Cells[1, colNames.Count]];
                    titleRange.Interior.Color = Color.FromArgb(204, 232, 207);

                    Microsoft.Office.Interop.Excel.Range sheetRange = null;

                    int rowNumber = tb.Rows.Count;
                    int columnNumber = tb.Columns.Count;

                   
                    if (rowNumber>0 && columnNumber>0)
                    {
                        object[,] objData = new object[rowNumber, columnNumber];
                        for (int r = 0; r < rowNumber; r++)
                        {   
                            for (int c = 0; c < columnNumber; c++)
                            {
                                objData[r, c] = tb.Rows[r][c].ToString().Trim();
                            }
                        }

                        sheetRange = excelSheet.Range[excelSheet.Cells[2, 1], excelSheet.Cells[rowNumber + 1, columnNumber]];
                        sheetRange.Value2 = objData;

                    }
                }

                excelBook.SaveAs(excelFileFullPath, Missing.Value, Missing.Value, Missing.Value, Missing.Value,
                   Missing.Value, XlSaveAsAccessMode.xlNoChange, Missing.Value, Missing.Value, Missing.Value,
                   Missing.Value, Missing.Value);

                excelBook.Close();

            }
            catch (System.Exception ex)
            {
                MessageBox.Show(ex.ToString());
            	
            }
            finally
            {
                closeExcel(excelAp);
            }

        }
    }
}

```
注意关闭excel进程：

```
/// <summary>
        ///  关闭excel进程
        /// </summary>
        /// <param name="excelAp">需要关闭的excel进程</param>
        public static void closeExcel(Microsoft.Office.Interop.Excel.Application excelAp)
        {
            if (excelAp != null)
            {              
                excelAp.Quit();
                int lpdwProcessId;
                GetWindowThreadProcessId(new IntPtr(excelAp.Hwnd), out lpdwProcessId);
                System.Diagnostics.Process.GetProcessById(lpdwProcessId).Kill();
            }
        }


```

这里单纯excelAp.Quit()是不够的，用到了P/Invoke调用，利用Windows API GetWindowThreadProcessId获取excel进程ID，然后C#杀死它。



## JSON文件解析 ##

假设有一个配置数据库地址的json配置文件：

```
{
  "Server": "1.1.1.1",
  "Port": "5518",
  "DataBase": "model_lib_db",
  "User": "root",
  "Password": "db10 "
}

```

怎么对它进行解析和读写？
我选择应用第三方库Json.net。从官网 www.newtonsoft.com/json下载下来后，
Bin目录是相应平台的dll
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcwMTIzMTQzOTA3ODUy)

在引用中添加这个程序集：
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcwMTIzMTQzOTM0Mzk5)


建立一个与json配置文件对应的json对象：

```
class MQAddressJsonObject
    {
        public MQAddressJsonObject()
        {
            server = "";
            port = "";
            dataBase = "";
            user = "";
            password = "";
        }

        public MQAddressJsonObject(string server,string port,string dataBase,string user,string password)
        {
            this.server=server;
            this.port=port;
            this.dataBase=dataBase;
            this.user=user;
            this.password=password;
        }

        private string server;
        public string Server
        {
            get { return server; }
            set { server = value; }
        }
        private string port;
        public string Port
        {
            get { return port; }
            set { port = value; }
        }
        private string dataBase;
        public string DataBase
        {
            get { return dataBase; }
            set { dataBase = value; }
        }
        private string user;
        public string User
        {
            get { return user; }
            set { user = value; }
        }
        private string password;
        public string Password
        {
            get { return password; }
            set { password = value; }
        }

        public string toConnectDBStr()
        {
            StringBuilder connStr = new StringBuilder();

            connStr.Append("server=").Append(server).Append(";");
            connStr.Append("user=").Append(user).Append(";");
            connStr.Append("database=").Append(dataBase).Append(";");
            connStr.Append("port=").Append(port).Append(";");
            connStr.Append("password=").Append(password).Append(";");
            return connStr.ToString();
        }
    }


```

对解析过程的封装：

```
class MQModifyAddressConfig
    {
        private static string configPath = "";
        private static string addressConfigFile = "addressConfig.json";
        private static JObject addressJson = null;

        private static void readConfigDir()
        {
            if (configPath.Equals(""))
            {
                configPath = System.Environment.CurrentDirectory + "\\" + addressConfigFile;
            }
        }

        private static void readAddressJson()
        {            
            using (StreamReader reader = new StreamReader(configPath))
            {
                string exportJsonStr = reader.ReadToEnd();
                addressJson = JsonConvert.DeserializeObject(exportJsonStr) as JObject;
            }
        }

        private static void writeAddressJson(string jsonStr)
        {
            using(FileStream jsonFile=new FileStream(configPath,FileMode.Create))
            using(StreamWriter writer = new StreamWriter(jsonFile))
            {
                writer.Write(jsonStr);
            }
        }

        public static void saveAddressInfo(MQAddressJsonObject addressJsonObj)
        {
            readConfigDir();
            string res=JsonConvert.SerializeObject(addressJsonObj, Formatting.Indented);
            writeAddressJson(res);
        }


        public static MQAddressJsonObject parseAddressInfo()
        {
            MQAddressJsonObject addressJsonObj = new MQAddressJsonObject();
            try
            {
                readConfigDir();
                readAddressJson();

                addressJsonObj.Server = addressJson["Server"].ToString();
                addressJsonObj.Port = addressJson["Port"].ToString();
                addressJsonObj.DataBase = addressJson["DataBase"].ToString();
                addressJsonObj.User = addressJson["User"].ToString();
                addressJsonObj.Password = addressJson["Password"].ToString();
               
            }
            catch (System.Exception ex)
            {
                MessageBox.Show(ex.ToString());
            }

            return addressJsonObj;
        }
    }


```

使用方法：
获取Mysql的连接字符串：

```
sqlConnStr=MQModifyAddressConfig.parseAddressInfo().toConnectDBStr();
```

把一个MQAddressJsonObject保存在配置文件中：

```
MQModifyAddressConfig.saveAddressInfo(addrJsonObject);
```

## 界面交互 ##

### 主界面左侧导航菜单的实现 ###

我只需要简单的几个按钮，实现winform窗体的切换，我下载了参考资料1
http://download.csdn.net/detail/nanzhaonan/5403457的demo，已经满足了我的需求。
效果如下：
点击一个按钮的时候，会展开不同的二级菜单：
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcwMTIzMTU1MjMwNjky)

![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcwMTIzMTU1NTM0NDE5)

我做了少量的工作，把原作者的代码规范化了一下，并消除了重复代码：

```
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Drawing;
using System.Text;
using System.Windows.Forms;
using System.Collections;

namespace MQHelper
{
    public partial class FormMain : Form
    {
        public System.Windows.Forms.Panel ActivePanel = new Panel();
        public FormMain()
        {
            InitializeComponent();            
            initDefaultPanel();
            
        }
      

        private void panelReSize(object sender, EventArgs e)
        {
            for (int i = 0; i < ActivePanel.Controls.Count; i++)
            {
                ActivePanel.Controls[i].Left = (ActivePanel.Width - ActivePanel.Controls[i].Width) / 2;
            }
        }

        
        private void btn_SysSet_Click(object sender, EventArgs e)
        {

            if(ActivePanel.Name  == panel_sysSet.Name)
                return;   
            ActivePanel = panel_sysSet; 
            changeBtnPos(0);
            panelBringToFront(btn_sysSet.Name);
            panelReSize(this,e);
        }


        private void initDefaultPanel()
        {           
            btnToPanelMap.Add(MQCommonConst.BTN_IMPORT, panel_import);
         
            btnToPanelMap.Add(MQCommonConst.BTN_EXPORT, panel_export);

            //--系¦Ì统ª3设¦¨¨置?
            btnToPanelMap.Add(MQCommonConst.BTN_SYS_SET, panel_sysSet);

         
            btnToPanelMap.Add(MQCommonConst.BTN_POS_MANAGE, panel_postManage);


            foreach (Panel pe in btnToPanelMap.Values)
            {
                pe.Visible = false;
            }

            ActivePanel = panel_import;
            changeBtnPos(1);
            panelBringToFront(btn_import.Name);
            panelReSize(this, null);
        }


        private void panelBringToFront(string panelName)
        {
            Panel beShow=btnToPanelMap[panelName] as Panel;

            foreach (Panel pe in btnToPanelMap.Values)
            {
                if (pe != beShow)
                {
                    pe.Visible = false;
                    //pe.SendToBack();
                }
            }

            beShow.Visible = true;
            beShow.BringToFront();
            beShow.Dock = DockStyle.Fill;
        }


        private void changeBtnPos(int seq)
        {
            switch (seq)
            {
                case 0:
                    {
                        btn_sysSet.Dock = DockStyle.Top;
                        btn_import.SendToBack();
                        btn_import.Dock = DockStyle.Bottom;
                        btn_export.SendToBack();
                        btn_export.Dock = DockStyle.Bottom;
                        btn_posManage.SendToBack();
                        btn_posManage.Dock = DockStyle.Bottom;    
                        break;
                    }
                case 1:
                    {
                        btn_import.Dock = DockStyle.Top;
                        btn_sysSet.SendToBack();
                        btn_sysSet.Dock = DockStyle.Top;
                        btn_export.SendToBack();
                        btn_export.Dock = DockStyle.Bottom;
                        btn_posManage.SendToBack();
                        btn_posManage.Dock = DockStyle.Bottom;

                        break;
                    }
                case 2:
                    {
                        btn_export.Dock = DockStyle.Top;
                        btn_import.SendToBack();
                        btn_import.Dock = DockStyle.Top;
                        btn_sysSet.SendToBack();
                        btn_sysSet.Dock = DockStyle.Top;
                        btn_posManage.SendToBack();
                        btn_posManage.Dock = DockStyle.Bottom; 

                        break;
                    }
                case 3:
                    {
                        btn_posManage.Dock = DockStyle.Top; 
                        btn_export.SendToBack();
                        btn_export.Dock = DockStyle.Top;
                        btn_import.SendToBack();
                        btn_import.Dock = DockStyle.Top;
                        btn_sysSet.SendToBack();
                        btn_sysSet.Dock = DockStyle.Top;
                        break;
                    }
                  default:
                    break;
            }

        }


        private Hashtable btnToPanelMap = new Hashtable();  //--每个btton对应一个面版    
    }
}


```

用一个hashtable存放按钮和要展示的panel的对应关系：

```
private Hashtable btnToPanelMap = new Hashtable();  //--每个btton对应一个面版
```

每个button的名字设置成常量，表示系统设置、导入、导出、局点管理4个按钮：

```
class MQCommonConst
{
public const string BTN_SYS_SET = "btn_sysSet";
public const string BTN_IMPORT = "btn_import";
public const string BTN_EXPORT = "btn_export";
public const string BTN_POS_MANAGE = "btn_posManage";
}
```

```
private void initDefaultPanel()
        {
            //--导入
            btnToPanelMap.Add(MQCommonConst.BTN_IMPORT, panel_import);

             //--导出
            btnToPanelMap.Add(MQCommonConst.BTN_EXPORT, panel_export);

            //--系统设置
            btnToPanelMap.Add(MQCommonConst.BTN_SYS_SET, panel_sysSet);

            //--局点管理
            btnToPanelMap.Add(MQCommonConst.BTN_POS_MANAGE, panel_postManage);


            foreach (Panel pe in btnToPanelMap.Values)
            {
                pe.Visible = false; //--初始不可见
            }

            ActivePanel = panel_import;
            changeBtnPos(1);
            panelBringToFront(btn_import.Name);
            panelReSize(this, null);
        }


```

这里panel_import、panel_export、panel_sysSet、panel_postManage是指的二级菜单，类似：
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcwMTIzMTU1ODE0MTA3)

就是一个panel，包括几个panel，几个label。

这是改变button位置的函数，当点击不同的button时，button的位置进行重新排列：

```
private void changeBtnPos(int seq)
        {
            switch (seq)
            {
                case 0:
                    {
                        btn_sysSet.Dock = DockStyle.Top;
                        btn_import.SendToBack();
                        btn_import.Dock = DockStyle.Bottom;
                        btn_export.SendToBack();
                        btn_export.Dock = DockStyle.Bottom;
                        btn_posManage.SendToBack();
                        btn_posManage.Dock = DockStyle.Bottom;    
                        break;
                    }
                case 1:
                    {
                        btn_import.Dock = DockStyle.Top;
                        btn_sysSet.SendToBack();
                        btn_sysSet.Dock = DockStyle.Top;
                        btn_export.SendToBack();
                        btn_export.Dock = DockStyle.Bottom;
                        btn_posManage.SendToBack();
                        btn_posManage.Dock = DockStyle.Bottom;

                        break;
                    }
                case 2:
                    {
                        btn_export.Dock = DockStyle.Top;
                        btn_import.SendToBack();
                        btn_import.Dock = DockStyle.Top;
                        btn_sysSet.SendToBack();
                        btn_sysSet.Dock = DockStyle.Top;
                        btn_posManage.SendToBack();
                        btn_posManage.Dock = DockStyle.Bottom; 

                        break;
                    }
                case 3:
                    {
                        btn_posManage.Dock = DockStyle.Top; 
                        btn_export.SendToBack();
                        btn_export.Dock = DockStyle.Top;
                        btn_import.SendToBack();
                        btn_import.Dock = DockStyle.Top;
                        btn_sysSet.SendToBack();
                        btn_sysSet.Dock = DockStyle.Top;
                        break;
                    }
                  default:
                    break;
            }

        }


```
这是显示指定panel的函数，当点击不同的button时，传入button名，就从hashtable中取出要显示的panel，并隐藏不用显示的panel。

```
private void panelBringToFront(string panelName)
        {
            Panel beShow=btnToPanelMap[panelName] as Panel;

            foreach (Panel pe in btnToPanelMap.Values)
            {
                if (pe != beShow)
                {
                    pe.Visible = false;
                    //pe.SendToBack();
                }
            }

            beShow.Visible = true;
            beShow.BringToFront();
            beShow.Dock = DockStyle.Fill;
        }

```
Panel重绘函数：

```
private void panelReSize(object sender, EventArgs e)
        {
            for (int i = 0; i < ActivePanel.Controls.Count; i++)
            {
                ActivePanel.Controls[i].Left = (ActivePanel.Width - ActivePanel.Controls[i].Width) / 2;
            }
        }

```



### 拖拽功能的实现 ###

下面实现一个小功能，从comBox下拉列表中选择模型大类，从左边的listBox中选择需要新增的模型，拖拽到右边的listBox中。相反，从右到左拖拽则把右边listBox中的内容删除。

![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcwMTIzMTYwMDE5OTM2)

下拉列表选择模型大类，主要是实现comBox的 SelectedIndexChanged事件：

```
private void comBox_deviceType_SelectedIndexChanged(object sender, EventArgs e)
        { 
            string value = comBox_deviceType.SelectedValue.ToString();
            bindModelNameData(value);
        }

```

通过传递选中的值，传给bindModelNameData函数，这个函数的功能是根据参数，从Mysql中查出想要的值，并放到右边的listBox中。

下面是实现拖拽功能的全部代码，左边的listBox名字是listBox_modelAll，表示全部模型；右边的listBox名字是listBox_selectedModel，表示选中的模型。
通过实现listBox的MouseDown、DragEnter、DragDrop事件来实现。

```
private void listBox_modelAll_MouseDown(object sender, MouseEventArgs e)
        {
            int iSelectedIndex = this.listBox_modelAll.SelectedIndex;
            if (iSelectedIndex < 0)
            {
                return;
            }
            string value = listBox_modelAll.SelectedValue.ToString();
            listBox_modelAll.DoDragDrop(value, DragDropEffects.Move);
        }

        private void listBox_modelAll_DragEnter(object sender, DragEventArgs e)
        {
            e.Effect = DragDropEffects.Move;
        }

        private void listBox_modelAll_DragDrop(object sender, DragEventArgs e)
        {
            string data = e.Data.GetData(typeof(string)) as string;  
            listBox_selectedModel.Items.Remove(data);
        }

        private void listBox_selectedModel_DragEnter(object sender, DragEventArgs e)
        {            
            e.Effect = DragDropEffects.Move;           
        }

        private void listBox_selectedModel_DragDrop(object sender, DragEventArgs e)
        {            
            string data = e.Data.GetData(typeof(string)) as string;           
            listBox_selectedModel.Items.Add(data);
        }
       
        private void listBox_selectedModel_MouseDown(object sender, MouseEventArgs e)
        {
            int iSelectedIndex = this.listBox_selectedModel.SelectedIndex;

            if (iSelectedIndex<0)
            {
                return;
            } 

            string value = listBox_selectedModel.SelectedItem.ToString();
            listBox_selectedModel.DoDragDrop(value, DragDropEffects.Move);
        }


```



### TextBox默认提示的实现 ###
C#的textBox控件没有能填写默认字符的属性，需要自己实现。

如果textBox的Text为“”，则当光标离开这个textBox显示默认提示字符：
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcwMTIzMTYwMjU5MTI4)

当光标进入时，等待输入：
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcwMTIzMTYwMzIzOTU3)

定义一个初始化函数：

```
     private void initTextBoxMsg()
        {
            tBox_postDetail.Text = "请输入300字符以内的局点描述，如地点、微模块数量等，可为空";
            tBox_postDetail.ForeColor = Color.Gray;
            tBox_postDetailHasText = false;
        }

```
实现textBox的Enter和Leave事件，定义一个成员变量tBox_postDetailHasText来判断文本框中是否有内容。

```
private void tBox_postDetail_Enter(object sender, EventArgs e)
        {
            if (tBox_postDetailHasText == false)
                tBox_postDetail.Text = "";

            tBox_postDetail.ForeColor = Color.Black;
        }

        //--textbox失去焦点  
        private void tBox_postDetail_Leave(object sender, EventArgs e)
        {
            if (tBox_postDetail.Text == "")
            {
                initTextBoxMsg();
            }
            else
                tBox_postDetailHasText = true;
        }

```

在窗体初始化的组件之后，初始化这个文本框：

```
public FormAddPost()
        {
            InitializeComponent();
            initTextBoxMsg();
        }

```




### 模型库连接测试的实现 ###

在操作导入导出之前，需要先判断是否能够连接到数据库。在界面上新增了一个Menu来实现。

![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcwMTIzMTYwNTA5MTMx)

显示测试信息的是richTextBox控件，实现一个函数来追加文本和设置颜色，rTextBox_connTest为这个richTextBox控件的名字：

```
private void logAppend(Color color, string text)
        {
            rTextBox_connTest.AppendText("\n");
            rTextBox_connTest.SelectionColor = color;
            rTextBox_connTest.AppendText(text);
        } 
```

用一个类来实现测试连接:

```
class MQConnectLibTest:ModelQuery
    {
        public bool doTest(out string errMsg)
        {
            bool isSuccess = false;
            errMsg = "";
            isSuccess = isLibOpen(out errMsg);

            if (isSuccess)
            {
                errMsg = "Connect model lib success";
            }

            return isSuccess;
        }

        private bool isLibOpen(out string errMsg)
        {
            errMsg = "";
            bool isOpen=false;
            MySqlConnection mySqlConn=null;  

            try
            {
                mySqlConn = getMysqlConn();
                mySqlConn.Open();

                if (mySqlConn.State == ConnectionState.Closed || mySqlConn.State == ConnectionState.Broken)
                {
                    isOpen=false;
                }else
                {
                    isOpen=true;
                }
            }
            catch (System.Exception ex)
            {
                errMsg = ex.ToString();
            }
            finally
            {
                mySqlConn.Close();
            }

            return isOpen;
        }
    }


```
参数是out类型的string，用来保存测试结果。测试方法就是打开一个数据库，如果不能打开，则捕获这个异常，并赋给out参数。
在窗体中的使用：

```
  private void FormConnectLibTest_Load(object sender, EventArgs e)
        {
            connectToDB();
        }


        private void connectToDB()
        {
            string errMsg;
            MQAddressJsonObject addrInfo=MQModifyAddressConfig.parseAddressInfo();
            logAppend(Color.Green, "Begin connect lib test......\n");
            logAppend(Color.Blue, "IP:"+addrInfo.Server+"  port:"+addrInfo.Port+" dataBase:"+addrInfo.DataBase);
                        
            MQConnectLibTest connTest=new MQConnectLibTest();
            bool isSuccess = connTest.doTest(out errMsg);

            if (!isSuccess)
            {
                logAppend(Color.Red, "\nResult:"+errMsg);
            }
            else
            {
                logAppend(Color.Green, "\nResult:" + errMsg);
            }            
        }


```

这个FormConnectLibTest，就是一个包含了richTextBox控件的窗体，MQModifyAddressConfig.parseAddressInfo()是从json配置文件中解析地址。


### 全选/反选功能的实现 ###

这里实现一个dataGridView的全选与反选功能：

![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcwMTIzMTYwNzE5NDc4)

全选和反选的两个函数：

```
private void selecteAllCheckBox()
        {
            int count = dataGridView_modelSet.Rows.Count;

            for (int i = 0; i < count; i++)
            {
                DataGridViewCheckBoxCell checkCell = dataGridView_modelSet.Rows[i].Cells["selectModel"] as DataGridViewCheckBoxCell;

                if (Convert.ToBoolean(checkCell.Value) == false)
                {
                    checkCell.Value = true;
                    checkCell.EditingCellFormattedValue = true;                   
                }
            }
        }


        private void disSelectAllCheckBox()
        {
            int count = dataGridView_modelSet.Rows.Count;

            for (int i = 0; i < count; i++)
            {
                DataGridViewCheckBoxCell checkCell = dataGridView_modelSet.Rows[i].Cells["selectModel"] as DataGridViewCheckBoxCell;

                if (Convert.ToBoolean(checkCell.Value) == true)
                {
                    checkCell.Value = false;
                    checkCell.EditingCellFormattedValue = false;   
                }
            }
        }
```

DataGridView中的第一列名字叫"selectModel"，类型是DataGridViewCheckBoxCell。
思路是遍历所有行，如果Convert.ToBoolean(checkCell.Value)为true，则把Value和EditingCellFormattedValue两个属性置反，反之同理。

要更改EditingCellFormattedValue属性的原因是，其他代码中有一处实际选中一行的时候，取的是EditingCellFormattedValue的值。

使用方式是实现checkBox_selectedAll控件的CheckedChanged事件：

```
private void checkBox_selectedAll_CheckedChanged(object sender, EventArgs e)
        {
            if (checkBox_selectedAll.Checked)
            {
                selecteAllCheckBox();                
            } 
            else
            {
                disSelectAllCheckBox();                
            }

            changeBtnExportEnabled();
        }


```
最后一个函数changeBtnExportEnabled()是更改导出按钮是否可用。

```
private void changeBtnExportEnabled()
        {
            int i = 0;
            int count = dataGridView_modelSet.Rows.Count;

            for (; i < count; i++)
            {
                DataGridViewCheckBoxCell checkCell = dataGridView_modelSet.Rows[i].Cells["selectModel"] as DataGridViewCheckBoxCell;
                Boolean flag = Convert.ToBoolean(checkCell.EditingCellFormattedValue);
                if (flag == true)         //查找被选择的数据行
                {
                    btn_export.Enabled = true;
                    break;
                }
            }

            if (i == count)
            {
                btn_export.Enabled = false;
            }
        }


```

### 进度条窗体的实现 ###

实现一个进度条，用来显示导入导出的进度：

![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcwMTIzMTYxMTE1MjIy)

更改进度条的值用到了观察者模式，在C#中就是事件与委托。
具体的应用，由于涉及到的代码太多，无法说清楚。这里应该主要关注的一点是，由于窗体的显示线程与更改值的线程可能不是在同一线程中，所以这个进度条窗体本身也要用到委托。

```
public partial class FormProgressBar : Form
    {
        public FormProgressBar()
        {
            InitializeComponent();
        }

        private static  int finishedExportNum;

        private delegate void SetPos(ProgressBarType type,int value, int maxValue);

        public void setProgressValue(ProgressBarType type,int value,int maxValue)
        { 
           
            if (this.InvokeRequired)
            {
                SetPos setPos = new SetPos(setProgressValue);
                this.Invoke(setPos, new object[] {type, value, maxValue });
            } 
            else
            {
               
                if (type==ProgressBarType.IMPORT)
                {
                    this.progressBar_show.Maximum = maxValue;
                    this.progressBar_show.Value = value;
                    double progRate = 100 * (double)value / maxValue;

                    this.label_value.Text = "Progress :" + progRate.ToString() + "%";
                    if (value == this.progressBar_show.Maximum)
                    {
                        Thread.Sleep(1500);
                        this.Close();
                    } 
                }else if (type==ProgressBarType.EXPORT)
                {

                    finishedExportNum++;

                    this.progressBar_show.Maximum = MQCommonConst.NEED_EXPORT_EXCELS_NUM;
                    this.progressBar_show.Value = finishedExportNum;
                    double progRate = 100 * (double)finishedExportNum / MQCommonConst.NEED_EXPORT_EXCELS_NUM;

                    this.label_value.Text = "Progress :" + progRate.ToString() + "%";
                    

                    if (finishedExportNum==MQCommonConst.NEED_EXPORT_EXCELS_NUM)
                    {
                        finishedExportNum = 0;
                        this.Close();
                    }

                }
                

            }
        }


```

具体的逻辑不用管，只要关注下面的代码：
这里在窗体内定义一个委托，用来更新进度条：

```
private delegate void SetPos(ProgressBarType type,int value, int maxValue);
```

这里判断是否要使用委托：

```
if (this.InvokeRequired)
  {
      SetPos setPos = new SetPos(setProgressValue);
      this.Invoke(setPos, new object[] {type, value, maxValue });
   } 

```

### 利用反射实现字段判空 ###

在界面中输入局点信息的时候，有些文本框的内容可以为空，有些不能为空。
![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcwMTI1MTExNDQxODE4)

如果硬编码每一个文本框，根据Text属性来判断，如果只有几个文本框还好，随着文本框数量的增加，这种方法不太可取。

另一种方法是，把判空延迟到插入数据库的时候处理。有一个局点Bean，用来存放界面中的文本框信息：

```
class MQPostInfoBean:MQBeanBase
    {
        public MQPostInfoBean()
        {
            postName = "";
            startPostTime = "";
            idcimVersion = "";
            collectorVersion = "";
            postDetail = "";
        }

        private string postName;
        public string PostName
        {
            get { return postName; }
            set { postName = value; }
        }
        private string startPostTime;
        public string StartPostTime
        {
            get { return startPostTime; }
            set { startPostTime = value; }
        }
        private string idcimVersion;
        public string IdcimVersion
        {
            get { return idcimVersion; }
            set { idcimVersion = value; }
        }
        private string collectorVersion;
        public string CollectorVersion
        {
            get { return collectorVersion; }
            set { collectorVersion = value; }
        }

        private string postDetail;
        public string PostDetail
        {
            get { return postDetail; }
            set { postDetail = value; }
        }

        public override string toInsertSql()
        {
            string sqlDelete = string.Format("delete from {0} where POSTNAME in ( '{1}' );\n",
                MQCommonConst.IDCIM_POSTINFO_TABLE, postName);
            string sql = sqlDelete + string.Format("insert into {0} (POSTNAME,STARTPOSTTIME,IDCIMVERSION,COLLECTORVERSION,POSTDETAIL) values"
                + "('{1}','{2}','{3}','{4}','{5}');\n",
                MQCommonConst.IDCIM_POSTINFO_TABLE, postName, startPostTime, idcimVersion, collectorVersion,postDetail);

            return sql;
        }
    }


```
把界面上的文本框存放到Bean中之后，插入到数据库之前，利用反射机制，遍历Bean的属性，进行判空：

```
private void checkLegalPostInfo()
        {
            Type type = postInfo.GetType();

            foreach (PropertyInfo pi in type.GetProperties())
             {
                 string value = pi.GetValue(postInfo, null) as string;
                 string name = pi.Name;         //获得属性的名字

                 if (!name.Equals(MQCommonConst.ADDPOST_EXCEPT_CHECK_NULL_PROP) && (value.Equals("") || value.Equals("System.Data.DataRowView")))
                 {
                     throw new MQFieldInfoNullException("局点字段："+name.ToLower()+"不能为空");
                 }
             }

            if (postIncludeModuleList.Count<=0)
            {
                throw new MQPostIncludeModuleZeroException("请选择局点模型，不能为0");
            }
        }



```

抛出的是自定义的异常：

```
class MQFieldInfoNullException : ApplicationException  
    {
        public MQFieldInfoNullException(string message) : base(message) { }  
  
        public override string Message  
        {  
            get  
            {  
                return base.Message;  
            }  
        } 
    }

class MQPostIncludeModuleZeroException : ApplicationException
    {

        public MQPostIncludeModuleZeroException(string message) : base(message) { }  
  
        public override string Message  
        {  
            get  
            {  
                return base.Message;  
            }  
        } 
    }


```
