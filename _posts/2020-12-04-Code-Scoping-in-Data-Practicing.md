---
title: Code Scoping in Data Practicing
author: MijazzChan
date: 2020-12-4 01:48:17 +0800
categories: [NotForBlog, Personal_Notes]
tags: [notes]
---

# Code Scoping in Data Practicing

> Personal Notes

## `cast()` in `spark scala`

Found useful when I tried to import or operate on a `dataFrame`. Source code is as follows

```scala
  /**
   * Casts the column to a different data type.
   * 
   *   // Casts colA to IntegerType.
   *   import org.apache.spark.sql.types.IntegerType
   *   df.select(df("colA").cast(IntegerType))
   *
   *   // equivalent to
   *   df.select(df("colA").cast("int"))
   * 
   *
   * @group expr_ops
   * @since 1.3.0
   **/
  def cast(to: DataType): Column = withExpr { Cast(expr, to) }

  /**
   * Casts the column to a different data type, using the canonical string representation
   * of the type. The supported types are: `string`, `boolean`, `byte`, `short`, `int`, `long`,
   * `float`, `double`, `decimal`, `date`, `timestamp`.
   * 
   *   // Casts colA to integer.
   *   df.select(df("colA").cast("int"))
   * 
   *
   * @group expr_ops
   * @since 1.3.0
   **/
  def cast(to: String): Column = cast(CatalystSqlParser.parseDataType(to))
```

`cast()` method can receive `String` of type, supporting  `string`, `boolean`, `byte`, `short`, `int`, `long`,`float`, `double`, `decimal`, `date`, `timestamp`.

it uses `CatalystSqlParser.parseDataType()` -> abstract class -> `AstBuilder.visitPrimitiveDataType()`

there are several primitive data types written in `case` clause. 

```scala
/**
   * Create a Spark DataType.
   */
  private def visitSparkDataType(ctx: DataTypeContext): DataType = {
    HiveStringType.replaceCharType(typedVisit(ctx))
  }

  /**
   * Resolve/create a primitive type.
   */
  override def visitPrimitiveDataType(ctx: PrimitiveDataTypeContext): DataType = withOrigin(ctx) {
    val dataType = ctx.identifier.getText.toLowerCase(Locale.ROOT)
    (dataType, ctx.INTEGER_VALUE().asScala.toList) match {
      case ("boolean", Nil) => BooleanType
      case ("tinyint" | "byte", Nil) => ByteType
      case ("smallint" | "short", Nil) => ShortType
      case ("int" | "integer", Nil) => IntegerType
      case ("bigint" | "long", Nil) => LongType
      case ("float", Nil) => FloatType
      case ("double", Nil) => DoubleType
      case ("date", Nil) => DateType
      case ("timestamp", Nil) => TimestampType
      case ("string", Nil) => StringType
      case ("char", length :: Nil) => CharType(length.getText.toInt)
      case ("varchar", length :: Nil) => VarcharType(length.getText.toInt)
      case ("binary", Nil) => BinaryType
      case ("decimal", Nil) => DecimalType.USER_DEFAULT
      case ("decimal", precision :: Nil) => DecimalType(precision.getText.toInt, 0)
      case ("decimal", precision :: scale :: Nil) =>
        DecimalType(precision.getText.toInt, scale.getText.toInt)
      case (dt, params) =>
        val dtStr = if (params.nonEmpty) s"$dt(${params.mkString(",")})" else dt
        throw new ParseException(s"DataType $dtStr is not supported.", ctx)
    }
  }
```

Going deep, this package is located right under the `spark.sql` core component => `Catalyst`, which provides `spark.sql` with 

+ Parser
+ Analyzer
+ Optimizer



## `extends App` in `Scala`

```scala
/**
 *  '''''It should be noted that this trait is implemented using the [[DelayedInit]]
 *  functionality, which means that fields of the object will not have been initialized
 *  before the main method has been executed.'''''
**/

/** The main method.
   *  This stores all arguments so that they can be retrieved with `args`
   *  and then executes all initialization code segments in the order in which
   *  they were passed to `delayedInit`.
   *  @param args the arguments passed to the main method
   */
  @deprecatedOverriding("main should not be overridden", "2.11.0")
  def main(args: Array[String]) = {
    this._args = args
    for (proc <- initCode) proc()
    if (util.Properties.propIsSet("scala.time")) {
      val total = currentTime - executionStart
      Console.println("[total " + total + "ms]")
    }
  }
```

Object which extends `App` will get a extend-chain with DelayInit, and a time-consumption output.

The `main()` method will also be overwritten, thus the whole class may become a main method.(Kind of like a lang-trick)

Plus, the `DelayedInit` is flagged outdated.



## `Temp View` of spark

`spark.sql('SQL STATEMENT')` can easily call sql executed in spark session. Make sure you create a `tempview` before you try to sql your dataframe.

+ TempView

  `createOrReplaceTempView()`

+ GlobalTempView

  `createOrReplaceGlobalTempView()` 

```scala
@throws[AnalysisException]
  def createGlobalTempView(viewName: String): Unit = withPlan {
    createTempViewCommand(viewName, replace = false, global = true)
  }

/**
   * Creates or replaces a global temporary view using the given name. The lifetime of this
   * temporary view is tied to this Spark application.
   *
   * Global temporary view is cross-session. Its lifetime is the lifetime of the Spark application,
   * i.e. it will be automatically dropped when the application terminates. It's tied to a system
   * preserved database `global_temp`, and we must use the qualified name to refer a global temp
   * view, e.g. `SELECT * FROM global_temp.view1`.
   *
   * @group basic
   * @since 2.2.0
   */
  def createOrReplaceGlobalTempView(viewName: String): Unit = withPlan {
    createTempViewCommand(viewName, replace = true, global = true)
  }
```

Try not use `create...()`. It throws `AnalysisException` when the view name has already been taken.

Also, mind that `global_temp.viewname` is **required** when accessing GlobalTemp.



## Column to List in Pyspark

```python
eg = [0 for _ in range(4)]
eg[0] = list(exampleSparkDataFrame.toPandas()['Number'])
eg[1] = exampleSparkDataFrame.select('Number').rdd.flatMap(lambda x: x).collect()
eg[2] = exampleSparkDataFrame.select('Number').rdd.map(lambda x: x[0]).collect()
eg[3] = [x[0] for x in exampleSparkDataFrame.select('Number').collect()]
```

