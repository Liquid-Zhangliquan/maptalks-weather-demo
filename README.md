### 背景

我们经常遇到直接传输gis数据到前端展示的时候，有时候数据量一稍微多点，传输速度就减慢，因为我们用于传输的json格式比较大。

### Geobuf介绍

Geobuf是一种用于地理数据的紧凑二进制编码。

Geobuf 几乎无损地将GeoJSON数据压缩到协议缓冲区中。单独使用GeoJSON的优点：

非常紧凑：通常使GeoJSON小6-8倍。

即使比较gzip大小，也要小2-2.5倍。

非常快速的编码和解码 - 甚至比原生JSON解析/字符串化更快。

可以容纳任何GeoJSON数据，包括具有任意属性的扩展。

轻松增量解析 - 在阅读时获取功能，而无需构建整个数据的内存表示。

部分读取 - 只读取实际需要的部分，跳过其余部分。

与Mapbox Vector Tiles不同，它的目标是几乎无损压缩数据集-无需平铺，投影坐标，展平几何或剥离属性。

“几乎”无损意味着坐标以小数点后的6位数精度（约10cm）编码(目前还不是无损压缩)

请注意，编码架构尚不稳定-它可能会随着得社区反馈并发现改进它的新方法而改变

### Geobuf压缩

本文中使用PostGIS进行压缩(geobuf有多种压缩方式)

```

select ST_AsGeobuf(sample,'geom') FROM (SELECT id,geom from public."California" ) as sample

```

### Geobuf解压

后端读取二进制传给前端，前端使用pbf和geobuf这两个库进行解压得Geojson

```

fetch("http://localhost:8081/geobuf/")

        .then(response => response.arrayBuffer())

        .then(buffer => {

          var vt = new Pbf(buffer);

          var geojson = geobuf.decode(vt);

          console.log(JSON.stringify(geojson))

});

```

![](https://user-gold-cdn.xitu.io/2019/9/17/16d3cf7121a2b66d?w=1920&h=881&f=png&s=1844710)

### 对比 



|  数据 | geojson |geosjon(gz) | geobuf(gz) |

| ------| ------  | ------     | ------     |

| 20w面 | 57.3MB  | 7.6MB      | 4.4MB      |

| 1w    | 2.83MB  | 380kb      | 218kb      |

参考资料：<br>

https://deck.gl/#/documentation/deckgl-api-reference/layers/tile-layer

https://github.com/mapbox/geobuf

