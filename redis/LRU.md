### LRU

* Redis LRU有个很重要的点，你通过调整每次回收时检查的采样数量，以实现**调整**算法的精度。这个参数可以通过以下的配置指令调整:

  ```
  maxmemory-samples 5
  ```

通过CONFIG SET maxmemory-samples 命令在生产环境上设置不同的采样大小是非常简单的。