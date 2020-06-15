# HBase

## 一、HBase实现自定义过滤器

### 1.1 参考网址

[hbase自定义过滤器]( https://blog.csdn.net/weixin_42660202/article/details/81019849 )

### 1.2 过程

1. 编写下列文件并保存为LineDataFilter.proto

```protobuf
/**
 * Licensed to the Apache Software Foundation (ASF) under one
 * or more contributor license agreements.  See the NOTICE file
 * distributed with this work for additional information
 * regarding copyright ownership.  The ASF licenses this file
 * to you under the Apache License, Version 2.0 (the
 * "License"); you may not use this file except in compliance
 * with the License.  You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
 
// This file contains protocol buffers that are used for filters
 
option java_package = "com.wl.filter";//生成java代码的包名
option java_outer_classname = "MyLineDataFilterProtos";//生成的类名
option java_generic_services = true;
option java_generate_equals_and_hash = true;
option optimize_for = SPEED;
 
// This file contains protocol buffers that are used for comparators (e.g. in filters)
 
message CustomLineDataFilter {
    required bytes value = 1;
}
```

2. 生成类文件

```shell
protoc.exe -I=D:/软件/protobuf/proto --java_out=D:/软件/protobuf/proto  D:/软件/protobuf/LineDataFilter.proto
```

3. 创建maven项目，并将生成的MyLineDataFilterProtos.java放在com.wl.filter的包下
4. 编写LineDataFilter类(该类名与文件中的message相同)

```java
package com.wl.filter;

import com.google.protobuf.InvalidProtocolBufferException;
import org.apache.hadoop.hbase.Cell;
import org.apache.hadoop.hbase.exceptions.DeserializationException;
import org.apache.hadoop.hbase.filter.Filter;
import org.apache.hadoop.hbase.filter.FilterBase;
import org.apache.hadoop.hbase.util.ByteStringer;
import org.apache.hadoop.hbase.util.Bytes;

import java.io.IOException;
import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;

public class CustomLineDataFilter extends FilterBase {

    private byte[] value;
    private List<Integer> rowkeys;
    private boolean filterRow = true;

    public CustomLineDataFilter(byte[] value) {
        this.value = value;
        this.rowkeys = Arrays.asList(new String(value).trim().split(",")).stream().map(Integer::valueOf).collect(Collectors.toList());
    }

    @Override
    public boolean filterRowKey(byte[] buffer, int offset, int length) throws IOException {
        byte[] rowkey = Arrays.copyOfRange(buffer, offset, offset + length);
        if (rowkeys.contains(Bytes.toInt(rowkey))) {
            filterRow = false;
        }
        return false;
    }

    public void reset() {
        this.filterRow = true;
    }

    public ReturnCode filterKeyValue(Cell cell) throws IOException {
        return filterRow ? ReturnCode.NEXT_ROW : ReturnCode.INCLUDE;
    }

    public byte[] toByteArray() {
        MyLineDataFilterProtos.CustomLineDataFilter.Builder builder = com.wl.filter.MyLineDataFilterProtos.CustomLineDataFilter.newBuilder();
        if (this.value != null) {
            builder.setValue(ByteStringer.wrap(this.value));
        }

        return builder.build().toByteArray();
    }

    public static Filter parseFrom(byte[] pbBytes) throws DeserializationException {
        com.wl.filter.MyLineDataFilterProtos.CustomLineDataFilter proto;
        try {
            proto = com.wl.filter.MyLineDataFilterProtos.CustomLineDataFilter.parseFrom(pbBytes);
        } catch (InvalidProtocolBufferException var3) {
            throw new DeserializationException(var3);
        }

        return new CustomLineDataFilter(proto.getValue().toByteArray());
    }
}
```

5. 打成jar包，并上传到hbase服务器的lib目录下(重启Hbase服务器)

   - 利用maven工具打jar包

   - 利用idea中提供的方式打jar包

     - empty：该方式打出的jar包中依赖的三方库是独立于自己的包结构的

     <img src="D:\软件\typora\img\empty方式的jar包.png" style="zoom:50%;" />

     - from module with dependencies：依赖的库以包结构被打入jar包

       ​    								  <img src="D:\软件\typora\img\from module方式的jar包.png" style="zoom:50%;" />

