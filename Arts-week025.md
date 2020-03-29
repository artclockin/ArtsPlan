# Arts-025

## 1.Algorithm

820. [ 单词的压缩编码](https://leetcode-cn.com/problems/short-encoding-of-words/)

给定一个单词列表，我们将这个列表编码成一个索引字符串 `S` 与一个索引列表 `A`。

例如，如果这个列表是 `["time", "me", "bell"]`，我们就可以将其表示为 `S = "time#bell#"` 和 `indexes = [0, 2, 5]`。

对于每一个索引，我们可以通过从字符串 `S` 中索引的位置开始读取字符串，直到 "#" 结束，来恢复我们之前的单词列表。

那么成功对给定单词列表进行编码的最小字符串长度是多少呢？



**示例：**

```
输入: words = ["time", "me", "bell"]
输出: 10
说明: S = "time#bell#" ， indexes = [0, 2, 5] 。
```



**提示：**

1. `1 <= words.length <= 2000`

2. `1 <= words[i].length <= 7`

3. 每个单词都是小写字母 。

   

**My Solution:**

```go
//use trie tree
type trieNode struct {
	child map[rune]*trieNode
	count    int
}


func NewTrieNode() *trieNode {
	return  &trieNode{
		count:    0,
		child: make(map[rune]*trieNode,  26),
	}
}
func (t *trieNode) get(val rune) *trieNode {
	if _,ok := t.child[val - 'a']; !ok {
		t.child[val - 'a'] = NewTrieNode()
		t.count++
	}
	return t.child[val - 'a']
}

func minimumLengthEncoding(words []string) int {
	trie := NewTrieNode()
	nodes := make(map[*trieNode]int)

	for i := 0; i < len(words); i++ {
		word := words[i]
		cur := trie
		for j := len(word) - 1; j >= 0; j-- {
			cur = cur.get(rune(word[j]))
		}
		nodes[cur] = i
	}

	ans := 0;
	for node := range nodes {
		if node.count == 0 {
			ans += len(words[nodes[node]]) + 1
		}
	}
	return ans;

}
```



## 2.Review

[Easy Searching with Elasticsearch](https://blogs.oracle.com/javamagazine/easy-searching-with-elasticsearch)

对需要文本搜索、时间序列搜索和聚合的应用程序而言，Elasticsearch是一个更好的选择。本文介绍了Elasticsearch high和low-level API使用，包括同步和异步模式，探讨了如何使用Elasticsearch Java API创建、更新、删除和搜索索引中的文档。

- Low-Level 同步CRUD API。客户端需要依赖ElasticSearch最小，需要完成创建JSON对象请求的和手动解析响应。内存有限情况下，这是唯一可用的方案。客户端依赖包：

  ```java
  <dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-client</artifactId>
    <version>7.0.0</version>
  </dependency>
  ```

  创建客户端

  ```java
  public static void main(String[] args) {
    try (RestClient client = RestClient.builder(
            new HttpHost("localhost", 9200, "http")).build()){
                CrudMethodsSynchronous scm = 
               new CrudMethodsSynchronous(
                   "catalog_item_low_level", client);
    }
  }
  ```

  创建索引

  ```java
  public void createCatalogItem(List<CatalogItem> items) {
    items.stream().forEach(e-> {
        
      Request request = new Request("PUT", 
              String.format("/%s/_doc/%d", 
                   getIndex(), e.getId()));
      try {
          request.setJsonEntity(
              getObjectMapper().writeValueAsString(e));
        
          getRestClient().performRequest(request);
        } catch (IOException ex) {
          LOG.warn("Could not post {} to ES", e, ex);
        }
    });
  }
  ```

  查询索引

  ```java
  public List<CatalogItem> findCatalogItem(String text) {
      Request request = new Request("GET", 
               String.format("/%s/_search", getIndex()));
         
      request.setJsonEntity(String.format(SEARCH, text));
      try {
          Response response = client.performRequest(request);
     if (response.getStatusLine().getStatusCode()==OK) {
             List<CatalogItem> catalogItems = 
                  parseResultsFromFullSearch(response);
            
             return catalogItems;
         } 
      } catch (IOException ex) {
          LOG.warn("Could not post {} to ES", text, ex);
      }
      return Collections.emptyList();
  }
  ```

  其他也类似的Restful API调用

- 异步调用

  ```java
  public void createCatalogItem(List<CatalogItem> items) {
    CountDownLatch latch = new CountDownLatch(items.size());
    ResponseListener listener = new ResponseListener() {
      @Override
      public void onSuccess(Response response) {
        latch.countDown();
      }
      @Override
      public void onFailure(Exception exception) {
        latch.countDown();
        LOG.error(
          "Could not process ES request. ", exception);
      }
    };
        
    itemsToCreate.stream().forEach(e-> {
      Request request = new Request(
                   "PUT", 
                   String.format("/%s/_doc/%d", 
                                 index, e.getId()));
        try {
          request.setJsonEntity(
              jsonMapper().writeValueAsString(e));
          client.performRequestAsync(request, listener);
        } catch (IOException ex) {
          LOG.warn("Could not post {} to ES", e, ex);
        }
      });
    try {
      latch.await(); //wait for all the threads to finish
      LOG.info("Done inserting all the records to the index");
    } catch (InterruptedException e1) {
      LOG.warn("Got interrupted.",e1);
    }
  }
  ```

  对于高级REST客户端，使用异步客户端要容易得多。

  

- High-Level REST Client，引用maven
  
  ```java
  <dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>
      elasticsearch-rest-high-level-client
    </artifactId>
    <version>7.4.2</version>
  </dependency>
  ```
  
  ```java
  try(RestHighLevelClient client = 
      new RestHighLevelClient(
          RestClient.builder(
              new HttpHost("localhost", 9200, "http")))) {
  
      CrudMethodsSynchronous scm = 
          new CrudMethodsSynchronous(
              "catalog_item_high_level",  client);
    
    //创建索引对象
    public void createCatalogItem(List<CatalogItem> items) {
    items.stream().forEach(e-> {
      IndexRequest request = new IndexRequest(index);
        try {
          request.id(""+e.getId());
          request.source(jsonMapper.writeValueAsString(e), 
                     XContentType.JSON);
          request.timeout(TimeValue.timeValueSeconds(10));
          IndexResponse response = client.index(request,  
                              RequestOptions.DEFAULT);
          if (response.getResult() == 
                    DocWriteResponse.Result.CREATED) {
            LOG.info("Added catalog item with id {} "
                     + "to ES index {}", 
                 e.getId(), response.getIndex());  
               
          } else if (response.getResult() == 
                    DocWriteResponse.Result.UPDATED) {
            LOG.info("Updated catalog item with id {} " +
             " to ES index {}, version of the " +
           "object is {} ", 
           e.getId(), response.getIndex(), 
           response.getVersion()); 
          } 
     
        } catch (IOException ex) {
          LOG.warn("Could not post {} to ES", e, ex);
        }
      }
     );
  }
   
  //查找索引
    public List<CatalogItem> findCatalogItem(String text) {
      try {
          SearchRequest request = new SearchRequest(index); 
          SearchSourceBuilder scb = new SearchSourceBuilder();
          SimpleQueryStringBuilder mcb = 
         QueryBuilders.simpleQueryStringQuery(text);
          scb.query(mcb); 
          request.source(scb);
           
          SearchResponse response = 
              client.search(request, RequestOptions.DEFAULT);
          SearchHits hits = response.getHits();
          SearchHit[] searchHits = hits.getHits();
          List<CatalogItem> catalogItems = 
              Arrays.stream(searchHits)
                    .filter(Objects::nonNull)
                    .map(e -> toJson(e.getSourceAsString()))
                    .collect(Collectors.toList());
           
          return catalogItems;
      } catch (IOException ex) {
          LOG.warn("Could not post {} to ES", text, ex);
      }
      return Collections.emptyList();
  }
  ```
  
  
  
- 异步调用

  ```java
  public PlainActionFuture<SearchResponse> 
                                   findItem(String text) {
                   
      SearchRequest request = new SearchRequest(getIndex()); 
      SearchSourceBuilder ssb = new SearchSourceBuilder();
      SimpleQueryStringBuilder mqb = 
               QueryBuilders.simpleQueryStringQuery(text);
      ssb.query(mqb); 
      request.source(ssb);
      
      PlainActionFuture<SearchResponse> future = 
                                new PlainActionFuture<>();
      client.searchAsync(request, 
                         RequestOptions.DEFAULT, future);
      return future;
  }
  ```

  异步api的最大优点是，在需要搜索结果之前，可以在工作线程中执行其他操作。

  

- 流数据写入ElasticSeatch

  ```java
  private void sendBatchToElasticSearch(
                   List<LineFromShakespeare> linesInBatch, 
        RestHighLevelClient client, 
        String indexName) throws IOException {
           
      BulkRequest request = new BulkRequest();
      linesInBatch.stream().forEach(l -> {
          try {
              request.add(new IndexRequest(indexName)
                     .id(l.getId())
                     .source(jsonMapper.writeValueAsString(l), 
               XContentType.JSON));
          
         } catch (JsonProcessingException e) {
             LOG.error("Problem mapping object {}", l, e);
         }});
      LOG.info("Sending data to ES");
      client.bulk(request, RequestOptions.DEFAULT);
  }
  
  public void loadData(String file, String index) 
                 throws IOException, URISyntaxException {
    Path filePath = Paths.get(
             ClassLoader.getSystemResource(file).toURI());
  
    List<String> errors = new ArrayList<>();
    List<LineFromShakespeare> lines = new ArrayList<>();
    final int maxLinesInBatch = 1000;
    try(RestHighLevelClient client = 
      new RestHighLevelClient(
        RestClient.builder(
         new HttpHost("localhost", 9200, "http")))) {
            
        Files.lines(file).forEach( e-> {
          try {
              LineFromShakespeare line = 
                  jsonMapper.readValue(e, LineFromShakespeare.class);
              //enrich...
              linesInBatch.add(line);
              if (linesInBatch.size() >= maxLinesInBatch) {
                  sendBatchToElasticSearch(linesInBatch, 
                                           client, indexName);
                 linesInBatch.clear();
              }
          } catch (IOException ex) {
            errors.add(e);
            linesInBatch.clear();
          }});
  
      if (linesInBatch.size() != 0) {
          sendBatchToElasticSearch(linesInBatch, 
                              client, indexName);
          linesInBatch.clear();
      }
    }
         
    LOG.info("Errors found in {} batches", errors.size());
  }
  ```

  


## 3.Tips
Golang中Sleep中参数类型需要转换为duration
```go
type MyDuration time.Duration

func main() {
	time.Sleep(time.Second * 3) //相当于3*1000 * Millisecond
	time.Sleep(time.Second * 3 * time.Duration(rand.Int()))
	time.Sleep(time.Second * time.Duration(MyDuration(3)))
	println("sleep done")
}
```


## 4.Share

分享一个技术文章：[Redis 性能优化的 13 条军规！史上最全](https://juejin.im/post/5e79702af265da570c75580a)

Redis 是单线程执行的特点，因此它对性能的要求更加苛刻，本文我们将通过13条优化手段，让 Redis 更加高效的运行。

- **缩短键值对的存储长度；**尽量的缩短键值对的存储长度，必要时要对数据进行序列化和压缩再存储，序列化我们可以使用 protostuff 或 kryo，压缩我们可以使用 snappy。
- **使用 lazy free（延迟删除）特性；**建议开启其中的 lazyfree-lazy-eviction、lazyfree-lazy-expire、lazyfree-lazy-server-del 等配置，这样就可以有效的提高主线程的执行效率。
- **设置键值的过期时间；**
- **禁用长耗时的查询命令；**

  - 决定禁止使用 keys 命令；
- 避免一次查询所有的成员，要使用 scan 命令进行分批的，游标式的遍历；
  - 通过机制严格控制 Hash、Set、Sorted Set 等结构的数据大小；
- 将排序、并集、交集等操作放在客户端执行，以减少 Redis 服务器运行压力；
  - 删除 (del) 一个大数据的时候，可能会需要很长时间，所以建议用异步删除的方式 unlink，它会启动一个新的线程来删除目标数据，而不阻塞 Redis 的主线程。 

- **使用 slowlog 优化耗时命令；**
  - `slowlog-log-slower-than` ：用于设置慢查询的评定时间
  - `slowlog-max-len` ：用来配置慢查询日志的最大记录数。
  - `slowlog get n` 来获取相关的慢查询日志
- **使用 Pipeline 批量操作数据；**
- **避免大量数据同时失效；**可以在过期时间的基础上添加一个指定范围的随机数。
- **客户端使用优化；**尽量使用 Pipeline 和 Redis 连接池
- **限制 Redis 内存大小；**
  - 配置项 `maxmemory `
  - **设置内存淘汰策略**
    - **noeviction**：不淘汰任何数据，当内存不足时，新增操作会报错，Redis 默认内存淘汰策略；
    - **allkeys-lru**：淘汰整个键值中最久未使用的键值；
    - **allkeys-random**：随机淘汰任意键值;
    - **volatile-lru**：淘汰所有设置了过期时间的键值中最久未使用的键值；
    - **volatile-random**：随机淘汰设置了过期时间的任意键值；
    - **volatile-ttl**：优先淘汰更早过期的键值。
    - **volatile-lfu**：淘汰所有设置了过期时间的键值中，最少使用的键值；
    - **allkeys-lfu**：淘汰整个键值中最少使用的键值。
- **使用物理机而非虚拟机安装 Redis 服务；**通过 `./redis-cli --intrinsic-latency 100` 命令查看延迟时间，尽可能在物理机上直接部署 Redis 服务器。
- **检查数据持久化策略；**
  -  3 种持久化的方式
    - RDB（Redis DataBase，快照方式）将某一个时刻的内存数据，以二进制的方式写入磁盘；
    - AOF（Append Only File，文件追加方式），记录所有的操作命令，并以文本的形式追加到文件中；
    - 混合持久化方式，Redis 4.0 之后新增的方式，混合持久化是结合了 RDB 和 AOF 的优点，在写入的时候，先把当前的数据以 RDB 的形式写入文件的开头，再将后续的操作命令以 AOF 的格式存入文件，这样既能保证 Redis 重启时的速度，又能减低数据丢失的风险。
  - `config get aof-use-rdb-preamble` 命令查询是否开启混合持久化
  - 使用命令 `config set aof-use-rdb-preamble yes`开启
  -  redis.conf 文件，把配置文件中的 `aof-use-rdb-preamble no` 改为 `aof-use-rdb-preamble yes`
- **禁用 THP 特性；**echo never >  /sys/kernel/mm/transparent_hugepage/enabled
- **使用分布式架构来增加读写速度。**
  - Redis 分布式架构
    - 主从同步
    - 哨兵模式。哨兵模式是对于主从功能的升级，但当主节点奔溃之后，无需人工干预就能自动恢复 Redis 的正常使用。
    - Redis Cluster 集群


