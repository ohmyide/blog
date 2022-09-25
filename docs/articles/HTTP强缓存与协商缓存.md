---
title: HTTP 强缓存与协商缓存
date: 2021-07-26 14:25:26
tags: 网络
---

## 缓存优先级：Cache-Control > expires > Etag > Last-Modified

## 强缓存
强缓存是根据返回头中的 Expires 或者 Cache-Control 两个字段来控制的，都是表示资源的缓存有效时间。

Expires 是 http 1.0 的规范，值是一个GMT 格式的时间点字符串，比如 Expires:Mon,18 Oct 2066 23:59:59 GMT 。这个时间点代表资源失效的时间，如果当前的时间戳在这个时间之前，则判定命中缓存。有一个缺点是，失效时间是一个绝对时间，如果服务器时间与客户端时间偏差较大时，就会导致缓存混乱。而服务器的时间跟用户的实际时间是不一样是很正常的，所以 Expires 在实际使用中会带来一些麻烦。

Cache-Control这个字段是 http 1.1 的规范，一般常用该字段的 max-age 值来进行判断，它是一个相对时间，比如 .Cache-Control:max-age=3600 代表资源的有效期是 3600 秒。并且返回头中的 Date 表示消息发送的时间，表示当前资源在 Date ~ Date +3600s 这段时间里都是有效的。不过我在实际使用中常常遇到设置了 max-age 之后，在 max-age 时间内重新访问资源却会返回 304 not modified ，这是由于服务器的时间与本地的时间不同造成的。当然 Cache-Control 还有其他几个值可以设置， 不过相对来说都很少用了：

no-cache：告诉浏览器、缓存服务器，不管本地副本是否过期，使用资源副本前，一定要【发请求】到源服务器进行副本有效性校验

服务器接收到带no-cache的请求后，依然判断资源是否变更，是则返回新内容，否则返回304

no-store直接禁止浏览器缓存数据，每次请求资源都会向服务器要完整的资源， 类似于 network 中的 disabled cache。

public 可以被所有用户缓存，包括终端用户和 cdn 等中间件代理服务器。

private 只能被终端用户的浏览器缓存。

如果 Cache-Control与 Expires 同时存在的话， Cache-Control 的优先级高于 Expires 。

## 协商缓存
协商缓存是由服务器来确定缓存资源是否可用。 主要涉及到两对属性字段，都是成对出现的，即第一次请求的响应头带上某个字, Last-Modified 或者 Etag，则后续请求则会带上对应的请求字段 If-Modified-Since或者 If-None-Match，若响应头没有 Last-Modified 或者 Etag 字段，则请求头也不会有对应的字段。

Last-Modified/If-Modified-Since 二者的值都是 GMT 格式的时间字符串， Last-Modified 标记最后文件修改时间， 下一次请求时，请求头中会带上 If-Modified-Since 值就是 Last-Modified 告诉服务器我本地缓存的文件最后修改的时间，在服务器上根据文件的最后修改时间判断资源是否有变化， 如果文件没有变更则返回 304 Not Modified ，请求不会返回资源内容，浏览器直接使用本地缓存。当服务器返回 304 Not Modified 的响应时，response header 中不会再添加的 Last-Modified 去试图更新本地缓存的 Last-Modified， 因为既然资源没有变化，那么 Last-Modified 也就不会改变；如果资源有变化，就正常返回返回资源内容，新的 Last-Modified 会在 response header 返回，并在下次请求之前更新本地缓存的 Last-Modified，下次请求时，If-Modified-Since会启用更新后的 Last-Modified。

Etag/If-None-Match， 值都是由服务器为每一个资源生成的唯一标识串，只要资源有变化就这个值就会改变。服务器根据文件本身算出一个哈希值并通过 ETag字段返回给浏览器，接收到 If-None-Match 字段以后，服务器通过比较两者是否一致来判定文件内容是否被改变。与 Last-Modified 不一样的是，当服务器返回 304 Not Modified 的响应时，由于在服务器上ETag 重新计算过，response header中还会把这个 ETag 返回，即使这个 ETag 跟之前的没有变化。

HTTP 中并没有指定如何生成 ETag，可以由开发者自行生成，哈希是比较理想的选择。

### 为什么要有 Etag
HTTP1.1 中 Etag 的出现主要是为了解决几个 Last-Modified 比较难解决的问题：

一些文件也许会周期性的更改，但是内容并不改变(仅仅改变的修改时间)，这个时候我们并不希望客户端认为这个文件被修改了，而重新 GET；

某些文件修改非常频繁，比如在秒以下的时间内进行修改，(比方说 1s 内修改了 N 次)，If-Modified-Since 能检查到的粒度是秒级的，使用 Etag 就能够保证这种需求下客户端在 1 秒内能刷新 N 次 cache。

某些服务器不能精确的得到文件的最后修改时间。

## 浏览器强刷原理
浏览器缓存分为：
- 验证性缓存
- 非验证性缓存

非验证性缓存：浏览器根据过期时间来判断是否使用缓存，如果在有效期内，直接从浏览器缓存中读取文件，不发生http请求，涉及到的 header 字段有 Cache-Control、expires、pragma

验证性缓存：给服务端发送请求时，在 header 里附带条件，服务端在处理请求时根据指定条件做出判断，如果符合条件则返回一个 304 状态码并返回空的 body ，浏览器在接收到 304 状态码后得知本地缓存依然有效，直接从本地缓存读取；如果条件为假则返回 200 状态码并返回指定资源。涉及到的 header 字段有 etag 、 last-modified

### 浏览器强刷原理
浏览器强刷时，浏览器在请求头里会加上如下字段： 
- catch-control: no-catch
- pragma: no-catch

通过一下字段，主动跳过了 非验证性缓存 和 验证性缓存，直接从服务端获取最新资源。