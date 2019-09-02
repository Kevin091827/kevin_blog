# 一，easyPoi

关于easyPoi的介绍，可以查看其官方文档[easyPoi](http://easypoi.mydoc.io/)

# 二，springboot2.x集成easyPoi实现excel数据导入到数据库

## 1. 注解导入

相关注解介绍：

- @Excel 作用到实体类字段上面,是对Excel一列的一个描述

- @ExcelCollection 表示一个集合,主要针对一对多的导出,比如一个老师对应多个科目,科目就可以用集合表示

- @ExcelEntity 表示一个继续深入导出的实体,但他没有太多的实际意义,只是告诉系统这个对象里面同样有导出的字段

- @ExcelIgnore 和名字一样表示这个字段被忽略跳过这个导导出

- @ExcelTarget 这个是作用于最外层的对象,描述这个对象的id,以便支持一个对象可以针对不同导出做出不同处理


## 2. 引入依赖

```xml
        <!-- 集成easypoi组件 .导出excel http://easypoi.mydoc.io/ -->
        <dependency>
            <groupId>cn.afterturn</groupId>
            <artifactId>easypoi-base</artifactId>
            <version>3.2.0</version>
        </dependency>
        <dependency>
            <groupId>cn.afterturn</groupId>
            <artifactId>easypoi-web</artifactId>
            <version>3.2.0</version>
        </dependency>
        <dependency>
            <groupId>cn.afterturn</groupId>
            <artifactId>easypoi-annotation</artifactId>
            <version>3.2.0</version>
        </dependency>
```

## 3. 注解实体类


需要被导入的excel的形式是：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190902081924889.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

所以需要注解的字段如下：

```java
@Data
public class TbCommpany implements Serializable {
    private Long id;

    @Excel(name = "companyName")
    private String companyName;
    @Excel(name = "companyLogo")
    private String companyLogo;
    @Excel(name = "gmtCreate",importFormat = "yyyyMMddHHmmss")
    private Date gmtCreate;
    @Excel(name = "gmtModify",importFormat = "yyyyMMddHHmmss")
    private Date gmtModify;
```
注意时间要进行格式转换

## 4. excel导入导出工具类

我自己写了个excel的工具类，但是本文中的示例我没有使用工具类中的方法，他默认也有一些关于导入导出的工具类

```java
public class ExcelUtils {

    public static void exportExcel(List<?> list, String title, String sheetName, Class<?> pojoClass, String fileName,
                                   boolean isCreateHeader, HttpServletResponse response) {
        ExportParams exportParams = new ExportParams(title, sheetName);
        exportParams.setCreateHeadRows(isCreateHeader);
        defaultExport(list, pojoClass, fileName, response, exportParams);
    }

    public static void exportExcel(List<?> list, String title, String sheetName, Class<?> pojoClass, String fileName,
                                   HttpServletResponse response) {
        defaultExport(list, pojoClass, fileName, response, new ExportParams(title, sheetName));
    }

    public static void exportExcel(List<Map<String, Object>> list, String fileName, HttpServletResponse response) {
        defaultExport(list, fileName, response);
    }

    private static void defaultExport(List<?> list, Class<?> pojoClass, String fileName, HttpServletResponse response,
                                      ExportParams exportParams) {
        Workbook workbook = ExcelExportUtil.exportExcel(exportParams, pojoClass, list);
        if (workbook != null)
            ;
        downLoadExcel(fileName, response, workbook);
    }

    private static void downLoadExcel(String fileName, HttpServletResponse response, Workbook workbook) {
        try {
            response.setCharacterEncoding("UTF-8");
            response.setHeader("content-Type", "application/vnd.ms-excel");
            response.setHeader("Content-Disposition", "attachment;filename=" + URLEncoder.encode(fileName, "UTF-8"));
            workbook.write(response.getOutputStream());
        } catch (IOException e) {
            // throw new NormalException(e.getMessage());
        }
    }

    private static void defaultExport(List<Map<String, Object>> list, String fileName, HttpServletResponse response) {
        Workbook workbook = ExcelExportUtil.exportExcel(list, ExcelType.HSSF);
        if (workbook != null)
            ;
        downLoadExcel(fileName, response, workbook);
    }

    public static <T> List<T> importExcel(String filePath, Integer titleRows, Integer headerRows, Class<T> pojoClass) {
        if (StringUtils.isBlank(filePath)) {
            return null;
        }
        ImportParams params = new ImportParams();
        params.setTitleRows(titleRows);
        params.setHeadRows(headerRows);
        List<T> list = null;
        try {
            list = ExcelImportUtil.importExcel(new File(filePath), pojoClass, params);
        } catch (NoSuchElementException e) {
            // throw new NormalException("模板不能为空");
        } catch (Exception e) {
            e.printStackTrace();
            // throw new NormalException(e.getMessage());
        }
        return list;
    }

    public static <T> List<T> importExcel(MultipartFile file, Integer titleRows, Integer headerRows,
                                          Class<T> pojoClass) {
        if (file == null) {
            return null;
        }
        ImportParams params = new ImportParams();
        params.setTitleRows(titleRows);
        params.setHeadRows(headerRows);
        List<T> list = null;
        try {
            list = ExcelImportUtil.importExcel(file.getInputStream(), pojoClass, params);
        } catch (NoSuchElementException e) {
            // throw new NormalException("excel文件不能为空");
        } catch (Exception e) {
            // throw new NormalException(e.getMessage());
            System.out.println(e.getMessage());
        }
        return list;
    }

}
```

## 5. 导入excel到数据库

controller层：

```java
    /**
     * 导入excel
     * @param file
     * @return
     */
    @PostMapping("/importExcel")
    public AjaxResult importExcel(@RequestParam("file") MultipartFile file) {
        return companyService.importExcel(file);
    }
```

service层：

```java
    /**
     * 导入excel数据到mysql，redis
     * @param file
     * @return
     */
    @Override
    public AjaxResult importExcel(MultipartFile file) {
        ImportParams importParams = new ImportParams();
        // 数据处理
        importParams.setHeadRows(1);
        importParams.setTitleRows(1);
        // 需要验证
        importParams.setNeedVerfiy(false);
        try {
            ExcelImportResult<TbCommpany> result = ExcelImportUtil.importExcelMore(file.getInputStream(), TbCommpany.class,
                    importParams);
            List<TbCommpany> commpanyList = result.getList();
            for (TbCommpany commpany : commpanyList) {
                log.info("从Excel导入数据到数据库的详细为 ：{}", JSONObject.toJSONString(commpany));
                //保存到mysql
                int i = tbCommpanyMapper.insertSelective(commpany);
                if(i < 1){
                    log.error("保存失败");
                    return new AjaxResult().error("保存失败");
                }
                //查出自增id
                TbCommpanyExample tbCommpanyExample = new TbCommpanyExample();
                tbCommpanyExample.createCriteria().andCompanyNameEqualTo(commpany.getCompanyName());
                long id = tbCommpanyMapper.selectByExample(tbCommpanyExample).get(0).getId();
                commpany.setId(id);
                //保存到redis
                redisUtils.sSet("company",JSONObject.parseObject(JSONObject.toJSONString(commpany)));
            }
            log.info("从Excel导入数据一共 {} 行 ", commpanyList.size());
        } catch (Exception e) {
            log.error("导入失败：{}", e.getMessage());
            return new AjaxResult().error("导入失败");
        }
        return new AjaxResult().ok("导入成功");
    }
```

## 6. 测试

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190902084238478.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

测试结果：

![在这里插入图片描述](https://img-blog.csdnimg.cn/201909020843320.png)