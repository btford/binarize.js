# What is binarize.js?
binarize.js is a JavaScript library that converts any variable, array or object into binary format. This library is useful when you want to send and receive complex objects (especially when they include TypedArray) in ArrayBuffer format over WebSocket, XHR2, etc.  

## Why not Protocol Buffers nor MessagePack?
As they are widely used and have wider adoption in terms of implementation, I consider binarize.js is just a temporary solution and recommend to use [Protocol Buffers](https://code.google.com/p/protobuf/) or [MessagePack](http://msgpack.org/) in stead. The reasons I did this are  

* Any of current JS implementation of Protocol Buffers and MessagePack don’t support TypedArray.
* I don’t have time to learn their spec.

So, it would be appreciated if someone could provide TypedArray support on those protocols :)  

# How to use binarize.js?
    var typed = new Float64Array([1, Number.MAX_VALUE, Number.MIN_VALUE]);  
    var object = {  
      name: 'Eiji Kitamura',  
      array: [1, 2, 3],  
      object: {  
        name: 'Eiji Kitamura',  
        hello: 'こんにちは',  
        typed: typed  
      }  
    };  
  
    // convert a JavaScript object into ArrayBuffer  
    var buffer = binarize.pack(object);  
  
    // retrieve original object from ArrayBuffer  
    var original = binarize.unpack(buffer);  

## Supported Types
    NULL:          0,  
    UNDEFINED:     1,  
    STRING:        2,  
    NUMBER:        3,  
    BOOLEAN:       4,  
    ARRAY:         5,  
    OBJECT:        6,  
    INT8ARRAY:     7,  
    INT16ARRAY:    8,  
    INT32ARRAY:    9,  
    UINT8ARRAY:    10,  
    UINT16ARRAY:   11,  
    UINT32ARRAY:   12,  
    FLOAT32ARRAY:  13,  
    FLOAT64ARRAY:  14  
  

# Formats

## Null
Null header will have no payload.  
  
     type     byte_length  
    +--------+----------------+  
    |0x0     |0x00            |  
    +--------+----------------+  

## Undefined
Undefined header will have no payload.  
  
     type     byte_length  
    +--------+----------------+  
    |0x1     |0x00            |  
    +--------+----------------+  

## Strings
Strings header will be followed by sequence of characters in Uint16Array.  
  
    ex) hello  
     type     byte_length      payload  
    +--------+----------------+----------------+----------------+  
    |0x2     |0x0a            |0x68 (h)        |0x65 (e)        |  
    +--------+-------+--------+-------+--------+-------+--------+  
    |0x6c (l)        |0x6c (l)        |0x6f (o)        |  
    +----------------+----------------+----------------+  

## Number
Number header will be followed by number in Float64Array.  
  
    ex) Number.MAX_VALUE  
     type     byte_length      payload  
    +--------+----------------+--------------------------------+  
    |0x3     |0x0e            |0x7fefffffffffffff              |  
    +--------+----------------+--------------------------------+  

## Boolean
Boolean header will followed by     0x1 when     true,     0x0 when     false.  
  
    ex) true  
     type     byte_length      payload  
    +--------+----------------+--------+  
    |0x4     |0x01            |0x1     |  
    +--------+----------------+--------+  

## Array
Array header will be followed by sequence of array elements.  
  
     type     length           byte_length  
    +--------+----------------+----------------+  
    |0x5     |0x03            |0x21            |  
    +--------+----------------+--------------------------------+  
    |0x3     |0x0e            +0x0000000000000001              |  
    +--------+----------------+--------------------------------+  
    |0x3     |0x0e            +0x0000000000000002              |  
    +--------+----------------+--------------------------------+  
    |0x3     |0x0e            +0x0000000000000003              |  
    +--------+----------------+--------------------------------+  

## Object
Object header will be followed by sequence of combinations of key in Strings type and value in arbitrary type.  
  
    ex) {‘name’: ‘Eiji Kitamura’, ‘hello’: ‘こんにちは’}  
     type     length           byte_length  
    +--------+----------------+----------------+  
    |0x6     |0x02            |0x42            |  
    +--------+----------------+----------------+----------------+  
    |0x2     |0x10            +0x006e (n)      |0x0061 (a)      |  
    +--------+-------+--------+-------+--------+----------------+  
    |0x006d (m)      |0x0065 (e)      |0x2     |0x1a            |  
    +--------+-------+--------+-------+--------+----------------+  
    |0x0045 (E)      |0x0069 (i)      |0x006a (j)      |...  
    +----------------+----------------+----------------+--------+  

## Int8Array
Int8Array header will be followed by sequence of 8bit interger values.  
  
    ex) 1  
     type     byte_length      payload  
    +--------+----------------+--------+  
    |0x07    |0x01            |0x1     |  
    +--------+----------------+--------+  

## Int16Array
Int16Array header will be followed by sequence of 16bit interger values.  
  
    ex) 16  
     type     byte_length      payload  
    +--------+----------------+----------------+  
    |0x08    |0x02            |0x10            |  
    +--------+----------------+----------------+  

## Int32Array
Int32Array header will be followed by sequence of 32bit interger values.  
  
    ex) 32  
     type     byte_length      payload  
    +--------+----------------+--------------------------------+  
    |0x09    |0x04            |0x0020                          |  
    +--------+----------------+--------------------------------+  

## Uint8Array
Uint8Array header will be followed by sequence of 8bit unsigned int values.  
  
    ex) 16  
     type     byte_length      payload  
    +--------+----------------+--------+  
    |0x10    |0x01            |0x10    |  
    +--------+----------------+--------+  

## Uint16Array
Uint16Array header will be followed by sequence of 16bit unsigned int values.  
  
    ex) 16  
     type     byte_length      payload  
    +--------+----------------+----------------+  
    |0x11    |0x02            |0x10            |  
    +--------+----------------+----------------+  

## Uint32Array
Uint32Array header will be followed by sequence of 32bit unsigned int values.  
  
    ex) 32  
     type     byte_length      payload  
    +--------+----------------+--------------------------------+  
    |0x12    |0x04            |0x0020                          |  
    +--------+----------------+--------------------------------+  

## Float32Array
Float32Array header will be followed by sequence of 32bit floating point values.  
  
    ex) 32  
     type     byte_length      payload  
    +--------+----------------+--------------------------------+  
    |0x13    |0x04            |0x0020                          |  
    +--------+----------------+--------------------------------+  

## Float64Array
Float64Array header will be followed by sequence of 64bit floating point values.  
  
    ex) 64  
     type     byte_length      payload  
    +--------+----------------+--------------------------------+  
    |0x14    |0x08            |0x0000000000000040              :  
    +--------+----------------+------+-------------------------+  
    :                                |  
    +--------------------------------+  