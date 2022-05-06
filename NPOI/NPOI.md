# NPOI

用于处理Excel,用法很简单  
首先，引入NuGet包，NPOI

其次将拓展方法复制进项目里边，拓展方法如下：

工厂

```csharp
    /// <summary>
    /// 工作薄工厂类,根据类型创建IWorkbook的实例
    /// </summary>
    public class WorkBookFactory
    {
        /// <summary>
        /// 创建一个工作簿对象
        /// </summary>
        /// <param name="type"></param>
        /// <returns></returns>
        public static IWorkbook Create(WorkBookType type)
        {
            IWorkbook workbook = null;
            switch (type)
            {
                case WorkBookType.xls:
                    workbook = new HSSFWorkbook();
                    break;
                case WorkBookType.xlsx:
                    workbook = new XSSFWorkbook();
                    break;
                default:
                    break;
            }
            return workbook;
        }

        /// <summary>
        /// 根据文件名获取工作簿对象
        /// </summary>
        /// <param name="file"></param>
        /// <returns></returns>
        public static IWorkbook GetWorkBook(string file)
        {
            string ext = Path.GetExtension(file).ToLower();
            if (ext == ".xls" && ext == ".xlsx")
            {
                throw new Exception("excel文件格式不正确");
            }
            using (FileStream readStram = File.OpenRead(file))
            {
                IWorkbook workbook = null;
                if (ext == ".xls")
                {
                    workbook = new HSSFWorkbook(readStram);
                }
                else
                {
                    workbook = new XSSFWorkbook(readStram);
                }
                return workbook;
            }
        }
    }
```

```csharp
    public static class NPOIHelper
    {
        //判断行是否为空行
        public static bool IsRowEmpty(this IRow row)
        {
            for (int c = row.FirstCellNum; c < row.LastCellNum; c++)
            {
                ICell cell = row.GetCell(c);
                if (cell != null && cell.CellType != CellType.Blank)
                {
                    return false;
                }
            }

            return true;
        }

        // 类型转换，当已明确需要一个数字返回时，fillZero为true，否则可能报错
        public static object GetCellValue(this ICell cell,bool fillZero)
        {
            if (cell == null)
            {
                return string.Empty;
            }

            switch (cell.CellType)
            {
                case CellType.Boolean:
                    return cell.BooleanCellValue;

                case CellType.Error:
                    return ErrorEval.GetText(cell.ErrorCellValue);

                case CellType.Formula:
                    switch (cell.CachedFormulaResultType)
                    {
                        case CellType.Boolean:
                            return cell.BooleanCellValue;

                        case CellType.Error:
                            return ErrorEval.GetText(cell.ErrorCellValue);

                        case CellType.Numeric:
                            if (DateUtil.IsCellDateFormatted(cell))
                            {
                                return cell.DateCellValue;
                            }
                            else
                            {
                                return cell.NumericCellValue;
                            }

                        case CellType.String:
                            string str = cell.StringCellValue;
                            if (!string.IsNullOrEmpty(str))
                            {
                                return str.ToString();
                            }
                            else
                            {
                                return string.Empty;
                            }

                        case CellType.Unknown:
                        case CellType.Blank:
                        default:
                            return string.Empty;
                    }

                case CellType.Numeric:
                    if (DateUtil.IsCellDateFormatted(cell))
                    {
                        return cell.DateCellValue;
                    }
                    else
                    {
                        return cell.NumericCellValue;
                    }

                case CellType.String:
                    string strValue = cell.StringCellValue;
                    return strValue.ToString().Trim();

                case CellType.Unknown:
                case CellType.Blank:
                    return fillZero ? 0 : string.Empty;

                default:
                    return string.Empty;
            }
        }

    }
```