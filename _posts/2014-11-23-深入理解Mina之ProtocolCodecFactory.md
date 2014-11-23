---
layout : life
title : "深入理解Mina之ProtocolCodecFactory"
category : 深入理解Mina
duoshuo: true
date : 2014-11-23
---

##编码器工厂

-----------

* 自定义编解码工厂：实现**ProtocolCodecFactory**接口
* 实现自定义编解码器：
 * 实现自动自定义的编码器:实现ProtocolEncoder接口
 * 实现自定义解码器:实现**ProtocolDecoder**接口
* 自定义的编解码器的重要性
 * 因为传输中往往不是通过一个字符串就可以传输所有的信息。
 * 我们传输的是自定义的协议包。并且能在应用程序和网络通信中存在对象和二进制流之间转化关系。
 * 所以我们需要结合业务编写自定义的编解码器.
* 常用的自定义协议的方法
 * 定长的方式 ： Aa,bb,cc,ok,no等这样的通信方式.
 * 定界符.helloworld|allpeople|.....|...  通过特殊的符号来区别消息.  这样方式会出现**粘包**，**半包**等现象.如：Hello     world|allpeople 带来了不正确消息，这样就应该丢弃数据.
* 自定义协议包
 * 包头：数据包的版本号，以及整个数据包(包头+包体) 长度
 * 包体：实际数据
 
-------------

##自定义协议包

---------------
{% highlight java %}
package com.shxz.protocal;

public class ProtocalPack {
	private int length;
	private byte flag;
	private String content;
	
	
	public ProtocalPack(byte flag,String content){
		this.flag=flag;
		this.content=content;
		int length=content==null?0:content.getBytes().length;
		this.length=5+length;
		//int是四个字节，flag是一个字节。总共五个字节。
		//length代表整个数据包
	}
	public int getLength() {
		return length;
	}
	public void setLength(int length) {
		this.length = length;
	}
	public byte getFlag() {
		return flag;
	}
	public void setFlag(byte flag) {
		this.flag = flag;
	}
	public String getContent() {
		return content;
	}
	public void setContent(String content) {
		this.content = content;
	}
	@Override
	public String toString(){
		StringBuffer sb=new StringBuffer();
		sb.append("length:").append(length);
		sb.append("flag:").append(flag);
		sb.append("content:").append(content);
		return sb.toString();
	}
}
{% endhighlight %}
-----------

##自定义编码器

--------------
{% highlight java %}
package com.shxz.protocal;

import java.nio.charset.Charset;

import org.apache.mina.core.buffer.IoBuffer;
import org.apache.mina.core.session.IoSession;
import org.apache.mina.filter.codec.ProtocolEncoderAdapter;
import org.apache.mina.filter.codec.ProtocolEncoderOutput;

public class ProtocalEncoder extends ProtocolEncoderAdapter{

	private final Charset charset;
	
	public ProtocalEncoder(Charset charset){
		this.charset=charset;
	}
	@Override
	public void encode(IoSession session, Object message,
			ProtocolEncoderOutput out) throws Exception {
		// TODO Auto-generated method stub
		ProtocalPack value=(ProtocalPack) message;
		
		IoBuffer buf=IoBuffer.allocate(value.getLength());
		buf.setAutoExpand(true);
		buf.putInt(value.getLength());
		buf.put(value.getFlag());
		if(value.getContent()!=null){
			buf.put(value.getContent().getBytes());
		}
		buf.flip();
		out.write(buf);
	}

}

{% endhighlight %}
-----------

##自定义解码器

------------

{% highlight java %}
package com.shxz.protocal;

import java.nio.charset.Charset;
import java.nio.charset.CharsetDecoder;

import org.apache.mina.core.buffer.IoBuffer;
import org.apache.mina.core.session.AttributeKey;
import org.apache.mina.core.session.IoSession;
import org.apache.mina.filter.codec.ProtocolDecoder;
import org.apache.mina.filter.codec.ProtocolDecoderOutput;

public class ProtocalDecoder implements ProtocolDecoder{

	private final AttributeKey CONTEXT=new AttributeKey(this.getClass(), "context");
	private final Charset charset;
	private int maxPacklength;
	
	public ProtocalDecoder(){
		this(Charset.defaultCharset());
	}
	public int getMaxPacklength() {
		return maxPacklength;
	}
	public void setMaxPacklength(int maxPacklength) {
		if(maxPacklength<0){
			throw new IllegalArgumentException("maxPacklength Exception:"+maxPacklength);
		}
		this.maxPacklength = maxPacklength;
	}
	public Charset getCharset() {
		return charset;
	}
	public ProtocalDecoder(Charset defaultCharset) {
		// TODO Auto-generated constructor stub
		this.charset=defaultCharset;
	}
	
	
	public Context getContext(IoSession session){
	
		Context context=(Context) session.getAttribute(CONTEXT);
		if(context==null){
			context=new Context();
			session.setAttribute("CONTEXT", context);
		}
		return context;
		
	}
	@Override
	public void decode(IoSession session, IoBuffer in, ProtocolDecoderOutput out)
			throws Exception {
		// TODO Auto-generated method stub
		final int packHeadlength=5;
		Context ctx=this.getContext(session);
		ctx.append(in);
		IoBuffer buf=ctx.getBuf();
		buf.flip();//缓冲区指针从）开始
		while(buf.remaining()>=packHeadlength){
			buf.mark();
			int length=buf.getInt();
			byte flag=buf.get();
			if(length<0||length>maxPacklength){
				buf.reset();
				break;
			}else if(length>=packHeadlength&&length-packHeadlength<=buf.remaining()){
				int old=buf.limit();
				buf.limit(buf.position()+length-packHeadlength);
				String content=buf.getString(ctx.getDecoder());
				ProtocalPack pakage=new ProtocalPack(flag,content);
				buf.limit(old);
				out.write(pakage);
			}else{//接收到的是一个半包
				buf.clear();
				break;
			}
			
		}
		if(buf.hasRemaining()){
			IoBuffer temp=IoBuffer.allocate(maxPacklength).setAutoExpand(true);
			temp.put(buf);
			temp.flip();
			temp.reset();
			buf.put(temp);
		}else{
			buf.reset();
		}
	}

	@Override
	public void finishDecode(IoSession session, ProtocolDecoderOutput out)
			throws Exception {
		// TODO Auto-generated method stub
		
	}

	@Override
	public void dispose(IoSession session) throws Exception {
		// TODO Auto-generated method stub
		Context ctx=(Context) session.getAttribute(CONTEXT);
		if(ctx!=null){
			session.removeAttribute(CONTEXT);
		}
	}
	private class Context{
		private final CharsetDecoder decoder;
		
		private IoBuffer buf;
		
		
		public void append(IoBuffer in){
			
			this.getBuf().put(in);
		}
		public void reset(){
			decoder.reset();
		}
		
		
		public IoBuffer getBuf() {
			return buf;
		}
		public void setBuf(IoBuffer buf) {
			this.buf = buf;
		}
		public CharsetDecoder getDecoder() {
			return decoder;
		}
		private Context(){
			decoder=charset.newDecoder();
			buf=IoBuffer.allocate(80).setAutoExpand(true);
		}
	}

}

{% endhighlight %}
-----------

##自定义编解码工厂：实现**ProtocolCodecFactory**接口

---------------

{% highlight java %}
package com.shxz.protocal;

import java.nio.charset.Charset;

import org.apache.mina.core.session.IoSession;
import org.apache.mina.filter.codec.ProtocolCodecFactory;
import org.apache.mina.filter.codec.ProtocolDecoder;
import org.apache.mina.filter.codec.ProtocolEncoder;

public class ProtocalFactory implements ProtocolCodecFactory{

	private final ProtocalEncoder encoder;
	private final ProtocalDecoder decoder;
	
	public ProtocalFactory(Charset charset){
		encoder=new ProtocalEncoder(charset);
		decoder=new ProtocalDecoder(charset);
	}
	
	@Override
	public ProtocolEncoder getEncoder(IoSession session) throws Exception {
		// TODO Auto-generated method stub
		return encoder;
	}

	@Override
	public ProtocolDecoder getDecoder(IoSession session) throws Exception {
		// TODO Auto-generated method stub
		return decoder;
	}

}

{% endhighlight %}
-----------





