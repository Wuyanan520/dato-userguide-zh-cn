# 入门

关于GraphLab Create的安装，请访问[dato.com安装指南页面]完成如下流程：

- 注册产品密钥
- 安装Python包`graphlab-create`

安装完成之后，你就可以将示例代码复制到终端（terminal）来运行。在整个指南中我们都假定你已经通过如下方式加载了graphlab这个包：

```python
import graphlab
```

有时你也可以通过如下方式将`graphlab`简写成`gl`：
```python
import graphlab as gl
```

### 快速入门

通过以下命令可以帮助你快速入门。
<table class="table table-bordered table-striped">
  <thead>
    <tr>
      <th>任务（Task）</th>
      <th>代码</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>导入和解析数据</td>
      <td><pre><code class="hljs python">url = 'http://s3.amazonaws.com/dato-datasets/millionsong/song_data.csv'<br/>songs = graphlab.SFrame.read_csv(url)</code></pre></td>
    </tr>
    <tr>
      <td>可视化</td>
      <td><pre><code class="hljs python">songs.show()</code></pre></td>
    </tr>
    <tr>
      <td>对列执行计算</td>
      <td><pre><code class="hljs python">songs['year'].mean()</code></pre></td>
    </tr>
    <tr>
      <td>转换列</td>
      <td><pre><code class="hljs python">songs['num_words'] = songs['title'].apply(lambda x: len(x.split(' ')))</code></pre></td>
    </tr>
    <tr>
      <td>聚合</td>
      <td><pre><code class="hljs python">songs.groupby('artist_name', {'total': graphlab.aggregate.COUNT})</code></pre></td>
    </tr>
    <tr>
      <td>创建模型</td>
      <td><pre><code class="hljs python">url = 'http://s3.amazonaws.com/dato-datasets/regression/Housing.csv'<br/>x = graphlab.SFrame.read_csv(url)<br/>m = graphlab.linear_regression.create(x, target='price')
      </code></pre></td>
    </tr>
  </tbody>
</table>
