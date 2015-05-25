# 代理之后的 Express

当在代理服务器之后运行 Express 时，请将应用变量 `trust proxy` 设置（使用 `app.set()`）为下述列表中的一项。

如果没有设置应用变量 `trust proxy`，应用将不会运行，除非 `trust proxy` 设置正确，否则应用会误将代理服务器的 IP 地址注册为客户端 IP 地址。

<table class="doctable" border="1">
<thead><tr><th>类型</th><th>Value</th></tr></thead>
<tbody>
<tr>
<td>Boolean</td>
<td>
<p>如果为 <code>true</code>，客户端 IP 地址为 <code>X-Forwarded-*</code> 头最左边的项。</p>
<p>如果为 <code>false</code>, 应用直接面向互联网，客户端 IP 地址从 <code>req.connection.remoteAddress</code> 得来，这是默认的设置。</p>
</td>
</tr>
<tr>
<td>IP 地址</td>
<td>
<p>IP 地址、子网或 IP 地址数组和可信的子网。下面是预配置的子网列表。</p>
<ul>
<li>loopback - <code>127.0.0.1/8</code>, <code>::1/128</code></li>
<li>linklocal - <code>169.254.0.0/16</code>, <code>fe80::/10</code></li>
<li>uniquelocal - <code>10.0.0.0/8</code>, <code>172.16.0.0/12</code>, <code>192.168.0.0/16</code>, <code>fc00::/7</code></li>
</ul>
<p>使用如下方式设置 IP 地址： </p>
<pre><code class="language-js">app.set('trust proxy', 'loopback') // 指定唯一子网
app.set('trust proxy', 'loopback, 123.123.123.123') // 指定子网和 IP 地址
app.set('trust proxy', 'loopback, linklocal, uniquelocal') // 指定多个子网
app.set('trust proxy', ['loopback', 'linklocal', 'uniquelocal']) // 使用数组指定多个子网</code></pre>
<p>当指定地址时，IP 地址或子网从地址确定过程中被除去，离应用服务器最近的非受信 IP 地址被当作客户端 IP 地址。</p>
</td>
</tr>
<tr>
<td>数字</td>
<td>
<p>将代理服务器前第 <code>n</code> 跳当作客户端。</p>
</td>
</tr>
<tr>
<td>函数</td>
<td>
<p>定制实现，只有在您知道自己在干什么时才能这样做。</p>
<pre><code class="language-js">app.set('trust proxy', function (ip) {
  if (ip === '127.0.0.1' || ip === '123.123.123.123') return true; // 受信的 IP 地址
  else return false;
})</code></pre>
</td>
</tr>
</tbody>
</table>

设置  `trust proxy` 为非假值会带来两个重要变化：

- 反向代理可能设置 `X-Forwarded-Proto` 来告诉应用使用 https 或简单的 http 协议。请参考 [req.protocol](http://expressjs.com/api.html#req.protocol)。
- [req.ip](http://expressjs.com/api.html#req.ip) 和 [req.ips](http://expressjs.com/api.html#req.ips) 的值将会由 `X-Forwarded-For` 中列出的 IP 地址构成。

`trust proxy` 设置由 [proxy-addr](https://www.npmjs.com/package/proxy-addr) 软件包实现，请参考其文档了解更多信息。