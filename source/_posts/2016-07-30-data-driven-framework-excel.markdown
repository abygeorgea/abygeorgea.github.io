---
layout: post
title: "Data Driven Framework - Excel"
date: 2016-07-30 05:49:22 +1000
comments: true
categories: 
- Data Driven Testing Framework
- Selenium WebDriver
- Excel
- CodeSnippets
keywords: Selenium , Specflow , Data Driven Testing , C# , XML
description: How to read from Excel for making a data driven framework 
---
In previous blog post, I have explained about how use XML for making a data driven framework for automation testing . It can be found [here]({{site.root}}blog/2016/07/29/data-driven-framework-xml/). I have also written about how to use jxl library for [reading from excel]({{site.root}}blog/2014/06/01/java-reading-a-specific-cell-in-excel/) and [writing into Excel]({{site.root}}blog/2014/06/01/java-writing-into-specific-cell-in-excel/).

Below is another code snippet to read all values of a row and save it into a hash map for accessing later during automation test. 

``` Java
public  HashMap GetAllDataForARow  (String sheet, int Row){
        HashMap DataMap = new HashMap();
        try {
            Workbook wrk1 =  Workbook.getWorkbook(new File(dataPath));

            //Obtain the reference to the first sheet in the workbook
            Sheet sheet1 = wrk1.getSheet(sheet);
            int x =0;


            Cell colArow1 , colArow2;
            do {
                colArow1 = sheet1.getCell(x,0);
                colArow2 = sheet1.getCell(x,Row);
                DataMap.put(colArow1.getContents(), colArow2.getContents());
                x=x+1;
            }while (colArow1.getContents() != "");

        }

        catch (BiffException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }catch (IndexOutOfBoundsException e){
        }
        return DataMap;

    }

```

