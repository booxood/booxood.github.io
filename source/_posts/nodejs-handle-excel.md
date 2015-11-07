title: Node.js Handle Excel
date: 2014-11-04 18:23
tags: Node.js
categories: Node.js
---

最近项目中需要用到xls文件的导入和导出，在github上找到几个的包：

> [js-xls](https://github.com/SheetJS/js-xls)  解析xls

> [js-xlsx](https://github.com/SheetJS/js-xlsx) 解析xlsx

> [xlgen](https://github.com/gutkyu/xlgen) 生成xls

> [excel-export-fast](https://github.com/elwerene/Node-Excel-Export) 生成xlsx

但其中xlgen这个包在生成xls文件时，如果生成的数据很多时，会很慢。3000行、100列的数据量需要11分钟，内存占用一直在170M左右，生成的文件大概4M，怎么都觉得不应该这么慢。又没找到其他合适的包，只能翻这个包的源码吧。

第一步想到的可能存在问题的地方可能是出在不停的打开文件写文件，不然不至于这么慢。通过看xlgen的代码，发现一次生成只会打开一次文件，就是第一次创建文件的时候。但写文件很频繁，先写头和一些xls文件结构相关的数据，再写cell，每一个cell拼装成一个buffer就写一次文件，问题就出在这里了。我把代码改成把所有的buffer组合在一起，最后再一次写入。buffer组合通过Buffer自带的.concat方法

    allBuf = []
    allBuf.push(buffer)
    allBuf = Buffer.concat(allBuf)

结果，需要17分钟，比之前还要慢。。。

百思不得其解，最终看Buffer模块的源码发现，Buffer.concat(list, [totalLength]) ，原来可以参入一个参数 总长度，如果不传入的话，则每次调用都要额外计算

    Buffer.concat = function(list, length) {
        if (!util.isArray(list))
            throw new TypeError('Usage: Buffer.concat(list[, length])');

        if (util.isUndefined(length)) {
            length = 0;
            for (var i = 0; i < list.length; i++)
                length += list[i].length;
        } else {
            length = length >>> 0;
        }

        ...

果断传入总长度，还是3000行100列，7秒就完成了！

总结：因为write buffer的event太多了，cpu处理不过来（单个CPU使用100%），事件都积压着（占用大量内存）。
