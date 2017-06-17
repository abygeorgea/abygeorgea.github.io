---
layout: post
title: "Java - Reading a specific cell in Excel"
date: 2014-06-01 21:04:52 +1000
comments: true
categories: CodeSnippets Java Excel
keywords: Java , Reading Excel
description: How to read a specific cell value from Excel
---

Below is a code snippet for reading a specific cell from Excel using Java.

It is done by using importing jxl jar files which can be found [here](https://sourceforge.net/projects/jxl/).

Import below in class file
```
import jxl.Cell;
import jxl.Sheet;
import jxl.Workbook;
import jxl.format.Colour;
import jxl.read.biff.BiffException;
import jxl.write.*;
```

Below is function for reading value from specific cell in Excel

``` Java Read From Excel based on Row and coulmn Number
 public  String readexcel(String sheet, int intRow, int intCol){
        try {

            //Create a workbook object from the file at specified location.
            //Change the path of the file as per the location on your computer.
          
            Workbook wrk1 =  Workbook.getWorkbook(new File(dataPath));

            //Obtain the reference to the first sheet in the workbook
            Sheet sheet1 = wrk1.getSheet(sheet);
            //Obtain reference to the Cell using getCell(int col, int row) method of sheet
			// Add " - 1" to both intRow , intCol depending how whether we consider excel start with row & column number as 0 or 1
            Cell colArow1 = sheet1.getCell(intRow , intCol );


            //Read the contents of the Cell using getContents() method, which will return
            //it as a String
            String strReturn = colArow1.getContents();

            return strReturn;

         /*  //Display the cell contents
           System.out.println("Contents of cell Col A Row 1: \""+str_colArow1 + "\"");
           System.out.println("Contents of cell Col B Row 1: \""+str_colBrow1 + "\"");
           System.out.println("Contents of cell Col A Row 2: \""+str_colArow2 + "\"");*/


        } catch (BiffException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }

        return sheet;
    }

```


Below is an example of how to read from a cell based on row number and Column NAME

``` Java Read From Excel Based On Column Name
 public  String GetDataBasedOnRowNumAndColName  (String sheet, String field_name, int Row){
        try {
            Workbook wrk1 =  Workbook.getWorkbook(new File(dataPath));

            //Obtain the reference to the first sheet in the workbook
            Sheet sheet1 = wrk1.getSheet(sheet);
            int x =0;
            int y =0;
            int Col=0;
            boolean FOUND = false;
           // Find corresponding Column number based on Name
            Cell colArow1 = sheet1.getCell(x,y);
            do {

                colArow1 = sheet1.getCell(x,y);
                if (colArow1.getContents().equalsIgnoreCase(field_name) ){
                    Col = colArow1.getColumn();
                    // System.out.println(Col);
                    FOUND = true;
                    break;
                }
                x=x+1;

            }while (colArow1.getContents() != "");
            if (FOUND)
            {
                colArow1 = sheet1.getCell(Col,Row-1);
                // System.out.println(colArow1.getContents());
                return colArow1.getContents();
            }
            else
            {
                return " ";
            }
        }

        catch (BiffException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }catch (IndexOutOfBoundsException e){
            return "";
        }

        return sheet;
    }
```