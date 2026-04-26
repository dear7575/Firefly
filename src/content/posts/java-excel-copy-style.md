---
title: ✨Excel 样式复制工具类
published: 2025-06-09
pinned: false
description: ""
tags: [Java, Excel]
category: 技术分享
draft: false
image: ./images/firefly1.avif
---

# Excel 样式复制工具类

```java
		/**
     * 上传Excel
     *
     * @param file
     * @return
     * @author lds
     * @date 2025/6/10 14:58
     */
		public Result uploadEditFieldExcel(MultipartFile file) {
        try {

            String sourceFilePath = "xxxxx";

            // 检查源文件是否存在
            File sourceFile = new File(sourceFilePath);
            if (!sourceFile.exists()) {
                return ActionResult.fail("源文件不存在");
            }

            // 使用样式复制方法：将新上传文件的数据复制到原文件中，保留原文件的样式
            copyExcelWithStyle(sourceFilePath, file);

            return ActionResult.success();
        } catch (IOException e) {
            return Result.fail("上传失败", e.getMessage());
        } catch (Exception e) {
            return Result.fail("处理文件时发生错误", e.getMessage());
        }
    }
  
    /**
     * 复制Excel样式
     *
     * @param sourceFilePath
     * @param destinationFile
     * @return
     * @author lds
     * @date 2025/6/10 14:58
     */
    public void copyExcelWithStyle(String sourceFilePath, MultipartFile destinationFile) throws IOException {
        // 读取现有的 Excel 文件
        Workbook existingWorkbook = WorkbookFactory.create(Files.newInputStream(Paths.get(sourceFilePath)));

        // 将前端上传的 Excel 文件转换为临时文件
        File tempFile = convertMultipartFileToTempFile(destinationFile);

        // 读取前端上传的 Excel 文件
        FileInputStream fileInputStream = new FileInputStream(tempFile);
        XSSFWorkbook workbook = new XSSFWorkbook(fileInputStream);

        // 复制上传的 Excel 中的数据和样式到现有的 Excel 文件
        copyExcelDataWithStyle(workbook, existingWorkbook);

        // 计算目标 Excel 文件中的公式
        evaluateFormulas(existingWorkbook);

        // 写回现有的 Excel 文件
        try (OutputStream outputStream = new FileOutputStream(sourceFilePath)) {
            existingWorkbook.write(outputStream);
        }

        // 删除临时文件
        tempFile.delete();
    }
  
    /**
     * 评估公式
     *
     * @param workbook
     * @author lds
     * @date 2024/4/18 11:15
     */
    private void evaluateFormulas(Workbook workbook) {
        FormulaEvaluator evaluator = workbook.getCreationHelper().createFormulaEvaluator();
        for (Sheet sheet : workbook) {
            for (Row row : sheet) {
                for (Cell cell : row) {
                    if (cell.getCellType() == CellType.FORMULA) {
                        evaluator.evaluateFormulaCell(cell);
                    }
                }
            }
        }
    }
  
    /**
     * 复制数据样式（保留目标文件的原有样式，只更新数据）
     *
     * @param sourceWorkbook 源数据（新上传的文件）
     * @param destWorkbook   目标文件（原有样式的文件）
     * @author lds
     * @date 2024/4/17 9:55
     */
    private void copyExcelDataWithStyle(XSSFWorkbook sourceWorkbook, Workbook destWorkbook) {
        int numSheets = Math.min(sourceWorkbook.getNumberOfSheets(), destWorkbook.getNumberOfSheets());

        for (int i = 0; i < numSheets; i++) {
            Sheet sourceSheet = sourceWorkbook.getSheetAt(i);
            Sheet destSheet = destWorkbook.getSheetAt(i);

            if (destSheet == null) {
                continue;
            }

            // 复制数据到现有的单元格，保留目标文件的样式
            for (Row sourceRow : sourceSheet) {
                int rowNum = sourceRow.getRowNum();
                Row destRow = destSheet.getRow(rowNum);

                if (destRow == null) {
                    destRow = destSheet.createRow(rowNum);
                }

                for (Cell sourceCell : sourceRow) {
                    int colNum = sourceCell.getColumnIndex();
                    Cell destCell = destRow.getCell(colNum);

                    if (destCell == null) {
                        destCell = destRow.createCell(colNum);
                    }

                    // 保存原有样式
                    CellStyle originalStyle = destCell.getCellStyle();

                    // 只复制数据，不复制样式
                    copyOnlyData(sourceCell, destCell);

                    // 如果目标单元格原来有样式，保持原样式
                    if (originalStyle != null && originalStyle.getIndex() != 0) {
                        destCell.setCellStyle(originalStyle);
                    }
                }
            }
        }
    }
  
    /**
     * 复制数据样式（保留源文件的部分样式，只更新数据）
     *
     * @param sourceWorkbook 源数据（新上传的文件）
     * @param destWorkbook   目标文件（原有样式的文件）
     * @author leihj
     * @date 2024/4/17 9:55
     */
    private void copyExcelDataWithStyle(XSSFWorkbook sourceWorkbook, Workbook destWorkbook) {
        int numSheets = Math.min(sourceWorkbook.getNumberOfSheets(), destWorkbook.getNumberOfSheets());

        for (int i = 0; i < numSheets; i++) {
            Sheet sourceSheet = sourceWorkbook.getSheetAt(i);
            Sheet destSheet = destWorkbook.getSheetAt(i);

            if (destSheet == null) {
                continue;
            }

            // 1. 删除目标工作表的合并区域（以源文件结构为准）
            for (int j = destSheet.getNumMergedRegions() - 1; j >= 0; j--) {
                destSheet.removeMergedRegion(j);
            }

            // 2. 复制源工作表的合并区域
            for (CellRangeAddress mergedRegion : sourceSheet.getMergedRegions()) {
                destSheet.addMergedRegion(mergedRegion);
            }

            // 3. 复制行高（以源文件行高为准）
            for (int rowIndex = 0; rowIndex <= sourceSheet.getLastRowNum(); rowIndex++) {
                Row sourceRow = sourceSheet.getRow(rowIndex);
                Row destRow = destSheet.getRow(rowIndex);
                if (sourceRow != null) {
                    // 创建行（如果不存在）
                    if (destRow == null) {
                        destRow = destSheet.createRow(rowIndex);
                    }
                    // 复制行高和隐藏属性
                    destRow.setHeight(sourceRow.getHeight());
                    destRow.setZeroHeight(sourceRow.getZeroHeight());
                }
            }

            // 4. 复制列宽（以源文件列宽为准）
            int maxColumnIndex = 0;
            for (Row row : sourceSheet) {
                if (row != null && row.getLastCellNum() > maxColumnIndex) {
                    maxColumnIndex = row.getLastCellNum();
                }
            }
            for (int columnIndex = 0; columnIndex < maxColumnIndex; columnIndex++) {
                int columnWidth = sourceSheet.getColumnWidth(columnIndex);
                destSheet.setColumnWidth(columnIndex, columnWidth);
                destSheet.setColumnHidden(columnIndex, sourceSheet.isColumnHidden(columnIndex));
            }

            // 5. 复制单元格数据（保留目标文件的样式）
            for (Row sourceRow : sourceSheet) {
                int rowNum = sourceRow.getRowNum();
                Row destRow = destSheet.getRow(rowNum);
                if (destRow == null) {
                    destRow = destSheet.createRow(rowNum);
                }

                for (Cell sourceCell : sourceRow) {
                    int colNum = sourceCell.getColumnIndex();
                    Cell destCell = destRow.getCell(colNum);
                    if (destCell == null) {
                        destCell = destRow.createCell(colNum);
                    }

                    // 保存目标单元格的原有样式
                    CellStyle originalStyle = destCell.getCellStyle();

                    // 仅复制源单元格的数据
                    copyOnlyData(sourceCell, destCell);

                    // 恢复目标单元格的原有样式
                    destCell.setCellStyle(originalStyle);
                }
            }
        }
    }
  
    /**
     * 只复制单元格数据，不复制样式
     *
     * @param sourceCell 源单元格
     * @param destCell   目标单元格
     * @author lds
     * @date 2024/4/17 9:56
     */
    private void copyOnlyData(Cell sourceCell, Cell destCell) {
        switch (sourceCell.getCellType()) {
            case STRING:
                destCell.setCellValue(sourceCell.getStringCellValue());
                break;
            case NUMERIC:
                if (org.apache.poi.ss.usermodel.DateUtil.isCellDateFormatted(sourceCell)) {
                    destCell.setCellValue(sourceCell.getDateCellValue());
                } else {
                    destCell.setCellValue(sourceCell.getNumericCellValue());
                }
                break;
            case BOOLEAN:
                destCell.setCellValue(sourceCell.getBooleanCellValue());
                break;
            case FORMULA:
                try {
                    destCell.setCellFormula(sourceCell.getCellFormula());
                } catch (Exception e) {
                    // 如果公式复制失败，则复制计算后的值
                    try {
                        destCell.setCellValue(sourceCell.getNumericCellValue());
                    } catch (Exception ex) {
                        destCell.setCellValue(sourceCell.getStringCellValue());
                    }
                }
                break;
            case BLANK:
                destCell.setBlank();
                break;
            default:
                // 处理其他类型
                try {
                    destCell.setCellValue(sourceCell.getStringCellValue());
                } catch (Exception e) {
                    destCell.setBlank();
                }
                break;
        }
    }
```
