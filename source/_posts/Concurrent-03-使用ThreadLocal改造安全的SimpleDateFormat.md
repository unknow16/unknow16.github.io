---
title: 使用ThreadLocal改造安全的SimpleDateFormat
date: 2018-01-28 21:01:31
tags: Concurrent
---

### SimpleDateFormat不安全原因
* parse 方法为什么不线程安全

1.有一个共享变量calendar，而这个共享变量的访问没有做到线程安全

2.parse方法生成CalendarBuilder，然后通过CalendarBuilder 设值到calendar，最后calendar.getTime();

* format方法为什么不线程安全

1.有一个共享变量calendar，而这个共享变量的访问没有做到线程安全

2.当使用format方法时，实际是给calent共享变量设置date值，然后调用subFormat将date转化成字符串

### 改造前

```
public class DateUtil {

    private final static SimpleDateFormat sdfyhm = new SimpleDateFormat(
            "yyyyMMdd");
            
    public synchronized static Date parseymdhms(String source) {
        try {
            return sdfyhm.parse(source);
        } catch (ParseException e) {
            e.printStackTrace();
            return new Date();
        }
    }

}
```

### 改造后

```
/**
 * 日期工具类(使用了ThreadLocal获取SimpleDateFormat,其他方法可以直接拷贝common-lang)
 * @author Niu Li
 * @date 2016/11/19
 */
public class DateUtil {

    private static Map<String,ThreadLocal<SimpleDateFormat>> sdfMap = new HashMap<String, ThreadLocal<SimpleDateFormat>>();

    private static Logger logger = LoggerFactory.getLogger(DateUtil.class);

    public final static String MDHMSS = "MMddHHmmssSSS";
    public final static String YMDHMS = "yyyyMMddHHmmss";
    public final static String YMDHMS_ = "yyyy-MM-dd HH:mm:ss";
    public final static String YMD = "yyyyMMdd";
    public final static String YMD_ = "yyyy-MM-dd";
    public final static String HMS = "HHmmss";

    /**
     * 根据map中的key得到对应线程的sdf实例
     * @param pattern map中的key
     * @return 该实例
     */
    private static SimpleDateFormat getSdf(final String pattern){
        ThreadLocal<SimpleDateFormat> sdfThread = sdfMap.get(pattern);
        if (sdfThread == null){
            //双重检验,防止sdfMap被多次put进去值,和双重锁单例原因是一样的
            synchronized (DateUtil.class){
                sdfThread = sdfMap.get(pattern);
                if (sdfThread == null){
                    logger.debug("put new sdf of pattern " + pattern + " to map");
                    sdfThread = new ThreadLocal<SimpleDateFormat>(){
                        @Override
                        protected SimpleDateFormat initialValue() {
                            logger.debug("thread: " + Thread.currentThread() + " init pattern: " + pattern);
                            return new SimpleDateFormat(pattern);
                        }
                    };
                    sdfMap.put(pattern,sdfThread);
                }
            }
        }
        return sdfThread.get();
    }

    /**
     * 按照指定pattern解析日期
     * @param date 要解析的date
     * @param pattern 指定格式
     * @return 解析后date实例
     */
    public static Date parseDate(String date,String pattern){
        if(date == null) {
            throw new IllegalArgumentException("The date must not be null");
        }
        try {
            return  getSdf(pattern).parse(date);
        } catch (ParseException e) {
            e.printStackTrace();
            logger.error("解析的格式不支持:"+pattern);
        }
        return null;
    }
    /**
     * 按照指定pattern格式化日期
     * @param date 要格式化的date
     * @param pattern 指定格式
     * @return 解析后格式
     */
    public static String formatDate(Date date,String pattern){
        if (date == null){
            throw new IllegalArgumentException("The date must not be null");
        }else {
            return getSdf(pattern).format(date);
        }
    }
}

```

##### 参考
https://www.jianshu.com/p/5675690b351e


