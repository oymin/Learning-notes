## 自定义 Generator 文件生成器

### 创建生成类

  1. 创建生成器

    > Generator generator = new Generator(getDataSource(), baseModelPackageName, baseModelOutputDir, modelPackageName, modelOutputDir);

  2. 调用 generate() 方法生成

### Generator 类

  - 引用对象

    - Dialect

    - MetaBuilder
      - setTypeMapping(typeMapping);
      - setRemovedTableNamePrefixes(removedTableNamePrefixes);
      - addExcludedTable(excludedTables);
      - setDialect(dialect);
      - List<TableMeta> tableMetas = metaBuilder.build();

### tableMeta

```
public class TableMeta {

	public String name;					// 表名
	public String remarks;				// 表备注
	public String primaryKey;			// 主键，复合主键以逗号分隔
	public List<ColumnMeta> columnMetas = new ArrayList<ColumnMeta>();	// 字段 meta

	// ---------

	public String baseModelName;		// 生成的 base model 名
	public String baseModelContent;		// 生成的 base model 内容

	public String modelName;			// 生成的 model 名
	public String modelContent;			// 生成的 model 内容

	// ---------

	public int colNameMaxLen = "Field".length();			// 字段名最大宽度，用于辅助生成字典文件样式
	public int colTypeMaxLen = "Type".length();				// 字段类型最大宽度，用于辅助生成字典文件样式
	public int colDefaultValueMaxLen = "Default".length();	// 字段默认值最大宽度，用于辅助生成字典文件样式
}
```

    - BaseModelGenerator Base model 生成器
      - generate(tableMetas);

    - ModelGenerator Model 生成器
      - generate(tableMetas);

    - MappingKitGenerator MappingKit 文件生成器
      - generate(tableMetas);

    - DataDictionaryGenerator DataDictionary 数据字典生成器
      - generate(tableMetas);
