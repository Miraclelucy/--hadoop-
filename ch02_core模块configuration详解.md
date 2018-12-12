Configuration类 
org.apache.hadoop.conf.Configuration
# 一、主要的成员变量
```java
private boolean quietmode = true;//1
private ArrayList<Object> resources = new ArrayList<Object>();//2
private boolean loadDefaults = true;//3
private static final CopyOnWriteArrayList<String> defaultResources =new CopyOnWriteArrayList<String>();//3
private Set<String> finalParameters = new HashSet<String>();//4
private Properties properties;//5
private Properties overlay;//6
private ClassLoader classLoader;//7
```
1. quietmode用来设置加载配置的模式，设置为true代表在加载配置信息的过程中不输出日志。
2. resources保存了所有通过addResource方法添加的Configuration对象的资源
3. loadDefaults确定是否加载默认资源，这些默认的资源存在defaultResources 中。
在hdfs中的默认资源是hdfs-default.xml和hdfs-site.xml;
在mapreduce中的默认资源是mapred-default.xml和mapred-site.xml;
4. 存放所有在配置文件中已经被申明为final的键值对
5. 存放配置文件解析后的健-值对
6. 存放通过set()方式改变的健-值对
7. 类加载器变量，可以加载指定的一个类，也可以用来加载资源

# 二、addResource方法：
```java
public void addResource(String name);//1
public void addResource(URL url);//2
public void addResource(Path file);//3 
public void addResource(InputStream in);//4
```
1. Classpath资源
2. 根据路径加载资源
3. 文件加载为资源
4. 一个输入流InputStream加载为资源

# 三、get*()和set*()方法：
Configuration中一共有21个get*方法，主要用来获取相应的配置资源：
这些配置信息可以是boolean/int/long等基本类型。也可以是一些Hadoop的常用类型，如类的信息(getClassByName,getClass,getClasses);String数组(getStingCollection);URL(getResource)。

最常用的是get()方法，它根据配置的健值获取值。实际调用了substituteVars方法：
```java
public String get(String name) {
  return substituteVars(getProps().getProperty(name));
}
````
substituteVars方法的源码：
```java
private static Pattern varPat = Pattern.compile("\\$\\{[^\\}\\$\u0020]+\\}");
private static int MAX_SUBST = 20;
private String substituteVars(String expr) {
  if (expr == null) {
    return null;
  }
  Matcher match = varPat.matcher("");
  String eval = expr;
  for(int s=0; s<MAX_SUBST; s++) {
    match.reset(eval);
    if (!match.find()) {
      return eval;
    }
    String var = match.group();
    var = var.substring(2, var.length()-1); // remove ${ .. }
    String val = null;
    try {
      val = System.getProperty(var);
    } catch(SecurityException se) {
      LOG.warn("Unexpected SecurityException in Configuration", se);
    }
    if (val == null) {
      val = getRaw(var);
    }
    if (val == null) {
      return eval; // return literal ${var}: var is unbound
    }
    // substitute
    eval = eval.substring(0, match.start())+val+eval.substring(match.end());
  }
  throw new IllegalStateException("Variable substitution depth too large: " 
                                  + MAX_SUBST + " " + expr);
}
```

# 四、Configurable接口
例如：org.apache.hadoop.mapred.SequenceFileInputFilter中的内部类RegexFilter的父类Filter实现了Configurable接口，那么RegexFilter这个类是可配置的。通过为RegexFilter这个类的对象传入一个Configuration实例，可以提供一些该对象工作时的配置信息。
```java
final private static String FILTER_REGEX = "sequencefile.filter.regex";
public static class RegexFilter extends FilterBase {
  private Pattern p;
  ...
  public void setConf(Configuration conf) {
    String regex = conf.get(FILTER_REGEX);
    if (regex==null)
      throw new RuntimeException(FILTER_REGEX + "not set");
    this.p = Pattern.compile(regex);
    this.conf = conf;
  }
 ...
}
```
实际调用该对象可以采用org.apache.hadoop.util.ReflectionUtils中的静态方法newInstance
```java
Configuration conf=new Configuration(); 
conf.set( "sequencefile.filter.regex","\\$\\{[^\\}\\$\u0020]+\\}");
ReflectionUtils.newInstance(RegexFilter.class, conf);
```




