---
layout: post
title: 我认为LengthFieldBasedFrameDecoder的正确使用方式
date: 2018-04-25 10:02:49
tags:
---

LengthFieldBasedFrameDecoder是Netty提供的便于编写一般基于长度的协议解码器，最近我发现在搜索引擎上搜到的使用方式代码如下
```java
    @Override
    protected Object decode(ChannelHandlerContext ctx, ByteBuf in) throws Exception {
        if (in == null) {
            return null;
        }
        //HEADER_SIZE 包头大小
        if (in.readableBytes() < HEADER_SIZE) {
            throw new Exception("readable bytes length less than header size");
        }
        int length = in.readInt();
        if (in.readableBytes() < length) {
            throw new Exception("body length less than length declared in header");
        }
        ByteBuf buf = in.readBytes(length);
        byte[] req = new byte[buf.readableBytes()];
        buf.readBytes(req);
        String body = new String(req, "UTF-8");
        Message msg = new Message(flag, type, body);
        return msg;
    }
```
关键点在可读数据不足时直接抛出异常是有点违背常理的，一般我们使用netty的ByteToMessageDecoder处理时代码如下
```
    @Override
    protected void decode(ChannelHandlerContext ctx, ByteBuf bufferIn, List<Object> out) throws Exception {
        if (in.readableBytes() < 4) {
            return null;
        }
        int readIndex = in.readerIndex();
        int length = in.readInt();
        if (in.readableBytes()+1 < length) {
            in.readerIndex(readIndex);
            return null;
        }
        ByteBuf buf = in.readBytes(length);
        byte[] req = new byte[buf.readableBytes()];
        buf.readBytes(req);
        String body = new String(req, "UTF-8");
        out.add(body);
    }
```
LengthFieldBasedFrameDecoder也继承自ByteToMessageDecoder主要有两个方法
```
    @Override
    protected final void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) throws Exception {
        Object decoded = decode(ctx, in);
        if (decoded != null) {
            out.add(decoded);
        }
    }
    protected Object decode(ChannelHandlerContext ctx, ByteBuf in) throws Exception {
        if (discardingTooLongFrame) {
            discardingTooLongFrame(in);
        }

        if (in.readableBytes() < lengthFieldEndOffset) {
            return null;
        }

        int actualLengthFieldOffset = in.readerIndex() + lengthFieldOffset;
        long frameLength = getUnadjustedFrameLength(in, actualLengthFieldOffset, lengthFieldLength, byteOrder);

        if (frameLength < 0) {
            failOnNegativeLengthField(in, frameLength, lengthFieldEndOffset);
        }

        frameLength += lengthAdjustment + lengthFieldEndOffset;

        if (frameLength < lengthFieldEndOffset) {
            failOnFrameLengthLessThanLengthFieldEndOffset(in, frameLength, lengthFieldEndOffset);
        }

        if (frameLength > maxFrameLength) {
            exceededFrameLength(in, frameLength);
            return null;
        }

        // never overflows because it's less than maxFrameLength
        int frameLengthInt = (int) frameLength;
        if (in.readableBytes() < frameLengthInt) {
            return null;
        }

        if (initialBytesToStrip > frameLengthInt) {
            failOnFrameLengthLessThanInitialBytesToStrip(in, frameLength, initialBytesToStrip);
        }
        in.skipBytes(initialBytesToStrip);

        // extract frame
        int readerIndex = in.readerIndex();
        int actualFrameLength = frameLengthInt - initialBytesToStrip;
        ByteBuf frame = extractFrame(ctx, in, readerIndex, actualFrameLength);
        in.readerIndex(readerIndex + actualFrameLength);
        return frame;
    }
```
可以看到LengthFieldBasedFrameDecoder的decode方法在可读数据不足是是会返回null的，
由此可见继承LengthFieldBasedFrameDecoder并重写decode方法在可读数据不足时抛出异常时不可取的