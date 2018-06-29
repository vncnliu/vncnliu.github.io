1.8中atomic依赖于Unsafe
在Unsafe的中
@CallerSensitive
    public static Unsafe getUnsafe() {
        Class<?> caller = Reflection.getCallerClass();
        if (!VM.isSystemDomainLoader(caller.getClassLoader()))
            throw new SecurityException("Unsafe");
        return theUnsafe;
    }
获取Unsafe的方法限制了调用类Unsafe中的方法基本都是通过offset(内存偏移量，应该是指变量相对于对象地址的内存偏移，用于获取内存地址）
直接操作内存，要注意其安全性

lazySet()就是在不需要让共享变量的修改立刻让其他线程可见的时候，以设置普通变量的方式来修改共享状态，可以减少不必要的内存屏障，从而提高程序执行的效率。

arrayIndexScale
javadoc
/**
     * Report the scale factor for addressing elements in the storage
     * allocation of a given array class.  However, arrays of "narrow" types
     * will generally not work properly with accessors like {@link
     * #getByte(Object, int)}, so the scale factor for such classes is reported
     * as zero.
     *
     * @see #arrayBaseOffset
     * @see #getInt(Object, long)
     * @see #putInt(Object, long, int)
     */
意思就是返回的是数组的scale factor（扩容因子，应该就是数组中每个元素的大小，如果大小不确定就返回0

主要作用还是用于计算偏移量以便于调用unsafe的方法直接操作内存

atomic中有一些累加器的实现类均利用Striped64类
主要思想就是每次添加的值都放到Striped64的cell列表里，这样在插入时需要锁的时候少，适用于更改多查询少的情况