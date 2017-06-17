---
layout: post
title: "Java - Writing Into specific cell in Excel"
date: 2014-06-10 22:24:30 +1000
comments: true
categories: CodeSnippets Java Excel
keywords: Java , Reading Excel
description: How to write into a specific cell value in Excel
---

In my previous blog post , I have mentioned how to read from an excel file using jxl jar files in Java. It can be found [here]({{site.root}}blog/2014/06/01/java-reading-a-specific-cell-in-excel/)

In this post, I will explain how to write into an excel using same library. Below example will update the excel cell content with the value passed and also update its formatting . The color of the cell will change depending on value we pass. We can use similar functions for updating any other cell format.

Below is the import section

```
import jxl.Cell;
import jxl.Sheet;
import jxl.Workbook;
import jxl.format.Colour;
import jxl.read.biff.BiffException;
import jxl.write.*;
```

Below is the function for writing into excel

``` Java
    public  void WriteDataIntoExcelCell (String sheet, String field_name, int Row, String input){
        try {

            Workbook wrk1 =  Workbook.getWorkbook(new File(dataPath));

            //Obtain the reference to the first sheet in the workbook
            Sheet sheet1 = wrk1.getSheet(sheet);
            int x =0;
            int y =0;
            int Col=0;
			// Find Column number from excel by iteration first row and comparing the names
            Cell colArow1 = sheet1.getCell(x,y);
            do {

                colArow1 = sheet1.getCell(x,y);
                if (colArow1.getContents().equalsIgnoreCase(field_name) ){
                    Col = colArow1.getColumn();
                    break;
                }
                x=x+1;

            }while (colArow1.getContents() != "");
			// write to file
            File exlFile = new File(dataPath);
            WritableWorkbook writableWorkbook = Workbook.createWorkbook(exlFile,wrk1);
            WritableSheet writableSheet = writableWorkbook.getSheet(sheet);
            //WritableCellFormat writableCell = writableWorkbook.getSheet(sheet).
			// Update cell content and format
            String Varcolour ;
            Label label;
            if (input.equalsIgnoreCase("PASS")){
                label = new Label(Col,Row,input,getCellFormat(Colour.GREEN));
            }
            else if (input.equalsIgnoreCase("FAIL"))
            {
                label = new Label(Col,Row,input,getCellFormat(Colour.RED));
            }
            else {
                label = new Label(Col,Row,input);
            }
            //Label label = new Label(Col,Row,input);

            writableSheet.addCell(label);

            writableWorkbook.write();
            writableWorkbook.close();


        }

        catch (BiffException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } catch (WriteException e) {
            e.printStackTrace();
        }



    }


    private static WritableCellFormat getCellFormat(Colour colour) throws WriteException {
        WritableFont cellFont = new WritableFont(WritableFont.TAHOMA, 10);
        WritableCellFormat cellFormat = new WritableCellFormat(cellFont);
        cellFormat.setBackground(colour);
        return cellFormat;
    }


```

