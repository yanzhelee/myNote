# JAVA IO 之BufferedInputStream BufferedOutputStream

# 有时间整理一下fill方法，并且把BufferedOutputStream源码解释翻译一下
## 1 BufferedInputStream

BufferedInputStream为另一个输入流添加一些功能，即缓冲输入以及支持mark和reset方法，在创建BufferedInputStream时会创建一个内部缓冲区数组。在读取或跳过流中的字节时，可根据需要从包含的输入流再次填充该内部缓冲区。一次填充多个字节。mark操作记录输入流中的某个节点，reset操作使得从包含的输入流中获取新字节之前，再次读取自最后一次mark操作后读取的所有字节。

主要字段：
<pre>
　  protected byte[] buf;//存储数据的内部缓冲区数组
　　protected int count;//缓冲区中有效字节的个数
　　protected int marklimit;//调用mark方法后,在后续调用reset方法失败之前允许的最大提前读取量
　　protected int markpos;//最后一次调用mark方法时pos字段的值
　　protected int pos;//缓冲区中的当前位置
</pre>

构造方法：
<pre>
　　BufferedInputStream(InputStream in)
　　　　创建一个BufferedInputStream并保存其参数,即输入流in,以便将来使用。
　　BufferedInputStream(InputStream in,int size)
　　　　创建具有指定缓冲区大小的BufferedInputStream并保存其参数,即输入流in,以便将来使用。
</pre>
主要方法：
<pre>
　　int available(): 返回缓存字节输入流中可读取的字节数
　　void close(): 关闭此缓存字节输入流并释放与该流有关的系统资源.
　　void mark(int readlimit): 在流中标记位置
　　boolean markSupported(): 测试该输入流是否支持mark和reset方法
　　int read(): 从缓冲输入流中读取一个字节数据
　　int read(byte[] b,int off,int len): 从缓冲输入流中将最多len个字节的数据读入到字节数组b中
　　long skip(long n): 从缓冲输入流中跳过并丢弃n个字节的数据
</pre>

BufferedInputStream的作用就是为其他输入流提供缓冲服务功能。创建BufferedInputSteam时我们会通过它的构造函数指定某个输入流作为参数，BufferedInputStream缓冲字节输入流。它作为FilterInputStream的一个子类，为传入的底层字节输入流提供缓冲功能，通过底层字节输入流读取字节 到自己的buffer中（内置的缓存字节数组），然后程序调用BufferedInputStream的read方法将buffer中的字节读取到程序中，当buffer中的字节被读取完之后，BufferedInputStream会从in中读取下一批数据块到buffer中，直到in中的数据被读取完毕，这样做的好处是提高读取的效率和减少打开存储介质的链接次数。


## 2 BufferedOutputStream

在BufferedOutputStream内部也提供了一个缓冲区，当缓冲区中的数据满了以后或者直接调用flush()方法就会把缓冲区中的数据写入到输出流。BufferedOutputStream比较简单直接看源码如下：
```java
package java.io;

/**
 * The class implements a buffered output stream. By setting up such
 * an output stream, an application can write bytes to the underlying
 * output stream without necessarily causing a call to the underlying
 * system for each byte written.
 *
 * @author  Arthur van Hoff
 * @since   JDK1.0
 */
public
class BufferedOutputStream extends FilterOutputStream {
    /**
     * The internal buffer where data is stored.
     */
    protected byte buf[];

    /**
     * The number of valid bytes in the buffer. This value is always
     * in the range <tt>0</tt> through <tt>buf.length</tt>; elements
     * <tt>buf[0]</tt> through <tt>buf[count-1]</tt> contain valid
     * byte data.
     */
    protected int count;

    /**
     * Creates a new buffered output stream to write data to the
     * specified underlying output stream.
     *
     * @param   out   the underlying output stream.
     */
    public BufferedOutputStream(OutputStream out) {
        this(out, 8192);
    }

    /**
     * Creates a new buffered output stream to write data to the
     * specified underlying output stream with the specified buffer
     * size.
     *
     * @param   out    the underlying output stream.
     * @param   size   the buffer size.
     * @exception IllegalArgumentException if size &lt;= 0.
     */
    public BufferedOutputStream(OutputStream out, int size) {
        super(out);
        if (size <= 0) {
            throw new IllegalArgumentException("Buffer size <= 0");
        }
        buf = new byte[size];
    }

    /** Flush the internal buffer */
    private void flushBuffer() throws IOException {
        if (count > 0) {
            out.write(buf, 0, count);
            count = 0;
        }
    }

    /**
     * Writes the specified byte to this buffered output stream.
     *
     * @param      b   the byte to be written.
     * @exception  IOException  if an I/O error occurs.
     */
    public synchronized void write(int b) throws IOException {
        if (count >= buf.length) {
            flushBuffer();
        }
        buf[count++] = (byte)b;
    }

    /**
     * Writes <code>len</code> bytes from the specified byte array
     * starting at offset <code>off</code> to this buffered output stream.
     *
     * <p> Ordinarily this method stores bytes from the given array into this
     * stream's buffer, flushing the buffer to the underlying output stream as
     * needed.  If the requested length is at least as large as this stream's
     * buffer, however, then this method will flush the buffer and write the
     * bytes directly to the underlying output stream.  Thus redundant
     * <code>BufferedOutputStream</code>s will not copy data unnecessarily.
     *
     * @param      b     the data.
     * @param      off   the start offset in the data.
     * @param      len   the number of bytes to write.
     * @exception  IOException  if an I/O error occurs.
     */
    public synchronized void write(byte b[], int off, int len) throws IOException {
        if (len >= buf.length) {
            /* If the request length exceeds the size of the output buffer,
               flush the output buffer and then write the data directly.
               In this way buffered streams will cascade harmlessly. */
            flushBuffer();
            out.write(b, off, len);
            return;
        }
        if (len > buf.length - count) {
            flushBuffer();
        }
        System.arraycopy(b, off, buf, count, len);
        count += len;
    }

    /**
     * Flushes this buffered output stream. This forces any buffered
     * output bytes to be written out to the underlying output stream.
     *
     * @exception  IOException  if an I/O error occurs.
     * @see        java.io.FilterOutputStream#out
     */
    public synchronized void flush() throws IOException {
        flushBuffer();
        out.flush();
    }
}
```

## 参考博文
http://www.cnblogs.com/xinhuaxuan/p/6062552.html