# SpringBoot 整合 ElasticSearch

## 1. 配置

### 1.1 pom 依赖

```xml
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-high-level-client</artifactId>
    <version>6.4.1</version>
</dependency>

<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-client</artifactId>
    <version>6.4.1</version>
</dependency>

<dependency>
    <groupId>org.elasticsearch</groupId>
    <artifactId>elasticsearch</artifactId>
    <version>6.4.1</version>
</dependency>
```

### 1.2 application.yml

```yml
server:
  port: ${port:40100}
spring:
  application:
    name: xc-search-service
xuecheng:
  elasticsearch:
    hostlist: ${eshostlist:127.0.0.1:9200} #多个结点中间用逗号分隔
  course:
    index: xc_course
    type: doc
    source_field: id,name,grade,mt,st,charge,valid,pic,qq,price,price_old,status,studymodel,teachmode,expires,pub_time,start_time,end_time
```

### 1.3 Configuration 配置类

```java
import org.apache.http.HttpHost;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestHighLevelClient;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class ElasticsearchConfig {

    @Value("${xuecheng.elasticsearch.hostlist}")
    private String hostlist;

    @Bean
    public RestHighLevelClient restHighLevelClient() {
        //解析hostlist配置信息
        String[] split = hostlist.split(",");
        //创建HttpHost数组，其中存放es主机和端口的配置信息
        HttpHost[] httpHostArray = new HttpHost[split.length];
        for (int i = 0; i < split.length; i++) {
            String item = split[i];
            httpHostArray[i] = new HttpHost(item.split(":")[0], Integer.parseInt(item.split(":")[1]), "http");
        }
        //创建RestHighLevelClient客户端
        return new RestHighLevelClient(RestClient.builder(httpHostArray));
    }

    //项目主要使用RestHighLevelClient，对于低级的客户端暂时不用
    @Bean
    public RestClient restClient() {
        //解析hostlist配置信息
        String[] split = hostlist.split(",");
        //创建HttpHost数组，其中存放es主机和端口的配置信息
        HttpHost[] httpHostArray = new HttpHost[split.length];
        for (int i = 0; i < split.length; i++) {
            String item = split[i];
            httpHostArray[i] = new HttpHost(item.split(":")[0], Integer.parseInt(item.split(":")[1]), "http");
        }
        return RestClient.builder(httpHostArray).build();
    }

}
```

---

## 2. java API 操作索引

```java
/**
 * 索引操作测试
 */
@SpringBootTest
@RunWith(SpringRunner.class)
public class TestIndex {

    @Autowired
    RestHighLevelClient restHighClient;
    @Autowired
    RestClient restClient;

    //删除索引库
    @Test
    public void testDeleteIndex() throws IOException {
        DeleteIndexRequest indexRequest = new DeleteIndexRequest("xc_course");
        IndicesClient indices = restHighClient.indices();
        DeleteIndexResponse deleteIndexResponse = indices.delete(indexRequest, RequestOptions.DEFAULT);
        boolean acknowledged = deleteIndexResponse.isAcknowledged();
        System.err.println(acknowledged);
    }

    //创建索引映射
    @Test
    public void testCreate() throws IOException {
        //创建索引请求对象，并设置索引名称
        CreateIndexRequest indexRequest = new CreateIndexRequest("xc_course");
        /**
         * 设置索引参数
         * number_of_shards：分片的数量
         * number_of_replicas：副本的数量
         */
        indexRequest.settings(Settings.builder().put("number_of_shards", 1).
                put("number_of_replicas", 0));

        Map<String, Object> mapping = new HashMap<>();
        Map<String, Object> properties = new HashMap<>();
        Map<String, Object> name = new HashMap<>();
        name.put("type", "text");
        name.put("analyzer", "ik_max_word");
        name.put("search_analyzer", "ik_smart");
        Map<String, Object> description = new HashMap<>();
        description.put("type", "text");
        description.put("analyzer", "ik_max_word");
        description.put("search_analyzer", "ik_smart");
        Map<String, Object> studymodel = new HashMap<>();
        studymodel.put("type", "keyword");
        Map<String, Object> price = new HashMap<>();
        price.put("type", "float");

        properties.put("name", name);
        properties.put("description", description);
        properties.put("studymodel", studymodel);
        properties.put("price", price);
        mapping.put("properties", properties);

        //创建索引操作客户端
        IndicesClient indices = restHighClient.indices();

        //设置映射
        indexRequest.mapping("doc", mapping);

        //创建响应对象
        CreateIndexResponse createIndexResponse = indices.create(indexRequest, RequestOptions.DEFAULT);

        //得到响应结果
        boolean acknowledged = createIndexResponse.isAcknowledged();
        System.err.println(acknowledged);
        System.err.println();
    }

    //判断索引是否存在
    @Test
    public void getIndex() throws IOException {
        //索引名称
        GetIndexRequest request = new GetIndexRequest();
        request.indices("xc_course");

        boolean exists = restHighClient.indices().exists(request, RequestOptions.DEFAULT);
        System.err.println(exists);
    }

}
```

---

## 3. java API 操作文档

```java
/**
 * 文档操作测试
 */
@SpringBootTest
@RunWith(SpringRunner.class)
public class TestDocument {

    @Autowired
    RestHighLevelClient highClient;
    @Autowired
    RestClient restClient;

    //添加文档
    @Test
    public void testAddDoc() throws IOException {
        //准备json数据
        Map<String, Object> jsonMap = new HashMap<>();
        jsonMap.put("name", "spring cloud实战");
        jsonMap.put("description", "本课程主要从四个章节进行讲解： 1.微服务架构入门 2.spring cloud基础入门 3.实战Spring Boot 4.注册中心eureka。");
        jsonMap.put("studymodel", "201001");
        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy‐MM‐dd HH:mm:ss");
        jsonMap.put("timestamp", dateFormat.format(new Date()));
        jsonMap.put("price", 5.6f);
        //索引请求对象
        IndexRequest indexRequest = new IndexRequest("xc_course", "doc");
        //指定索引文档内容
        indexRequest.source(jsonMap);
        //索引响应对象
        IndexResponse indexResponse = highClient.index(indexRequest, RequestOptions.DEFAULT);
        //获取响应结果
        DocWriteResponse.Result result = indexResponse.getResult();
        System.err.println(result);
    }

    //查询文档
    @Test
    public void getDoc() throws IOException {
        GetRequest getRequest = new GetRequest("xc_course", "doc", "K2ahTWwB6LxQ9ml8E6wm");
        GetResponse getResponse = highClient.get(getRequest, RequestOptions.DEFAULT);
        boolean exists = getResponse.isExists();
        Map<String, Object> sourceAsMap = getResponse.getSourceAsMap();
        System.err.println(exists);
        System.err.println(sourceAsMap);
    }

    //更新文档
    @Test
    public void updateDoc() throws IOException {
        UpdateRequest updateRequest = new UpdateRequest("xc_course", "doc", "K2ahTWwB6LxQ9ml8E6wm");
        Map<String, Object> map = new HashMap<>();
        map.put("name", "spring cloud实战(2)");
        updateRequest.doc(map);
        UpdateResponse updateResponse = highClient.update(updateRequest, RequestOptions.DEFAULT);
        RestStatus status = updateResponse.status();
        System.err.println(status);
    }

    //删除文档
    @Test
    public void deleteDoc() throws IOException {
        DeleteRequest deleteRequest = new DeleteRequest("xc_course", "doc", "K2ahTWwB6LxQ9ml8E6wm");
        DeleteResponse deleteResponse = highClient.delete(deleteRequest, RequestOptions.DEFAULT);
        DocWriteResponse.Result result = deleteResponse.getResult();
        RestStatus status = deleteResponse.status();
        System.err.println(result);
        System.err.println(status);
    }

}
```

---

## 4. DSL 搜索 java API

DSL (Domain Specific Language) 是 ES 提出的基于 json 的搜索方式，在搜索时传入特定的 json 格式的数据来完成不同的搜索需求。

DSL 比 URI 搜索方式功能强大，在项目中建议使用 DSL 方式来完成搜索。

```java
/**
 * DSL搜索查询测试
 */
@SpringBootTest
@RunWith(SpringRunner.class)
public class TestDSL {

    @Autowired
    RestHighLevelClient highClient;
    @Autowired
    RestClient restClient;

    //搜索type下的全部记录
    @Test
    public void testSearchAll() throws IOException {
        SearchRequest searchRequest = new SearchRequest("xc_course");
        searchRequest.types("doc");
        SearchSourceBuilder queryBuilder = new SearchSourceBuilder().query(QueryBuilders.matchAllQuery());
        //source源字段过滤
        queryBuilder.fetchSource(new String[]{"name", "studymodel"}, new String[]{});
        searchRequest.source(queryBuilder);
        SearchResponse searchResponse = highClient.search(searchRequest, RequestOptions.DEFAULT);
        printSearchResponse(searchResponse);
    }

    //分页查询
    @Test
    public void testSearchPaging() throws IOException {
        SearchRequest searchRequest = new SearchRequest("xc_course");
        searchRequest.types("doc");
        SearchSourceBuilder queryBuilder = new SearchSourceBuilder().query(QueryBuilders.matchAllQuery());
        //分页查询，设置起始下标，从0开始
        queryBuilder.from(0);
        //每页显示个数
        queryBuilder.size(2);
        //source源字段过滤
        queryBuilder.fetchSource(new String[]{"name", "studymodel"}, new String[]{});
        searchRequest.source(queryBuilder);
        SearchResponse searchResponse = highClient.search(searchRequest, RequestOptions.DEFAULT);
        printSearchResponse(searchResponse);
    }

    //Term Query为精确查询，在搜索时会整体匹配关键字，不再将关键字分词。
    @Test
    public void testTermQuery() throws Exception {
        SearchRequest searchRequest = new SearchRequest("xc_course");
        searchRequest.types("doc");
        SearchSourceBuilder queryBuilder = new SearchSourceBuilder().query(QueryBuilders.termQuery("name", "spring"));
        queryBuilder.fetchSource(new String[]{"name", "studymodel"}, new String[]{});
        searchRequest.source(queryBuilder);
        SearchResponse searchResponse = highClient.search(searchRequest, RequestOptions.DEFAULT);
        printSearchResponse(searchResponse);
    }

    //根据id精确匹配
    @Test
    public void testTermQueryByIds() throws IOException {
        SearchRequest searchRequest = new SearchRequest("xc_course");
        searchRequest.types("doc");
        String[] ids = {"1", "3"};
        List<String> idList = Arrays.asList(ids);
        SearchSourceBuilder queryBuilder = new SearchSourceBuilder().query(QueryBuilders.termsQuery("_id", idList));
        queryBuilder.fetchSource(new String[]{"name", "studymodel"}, new String[]{});
        searchRequest.source(queryBuilder);
        SearchResponse searchResponse = highClient.search(searchRequest, RequestOptions.DEFAULT);
        printSearchResponse(searchResponse);
    }

    //match Query即全文检索，它的搜索方式是先将搜索字符串分词，再使用各各词条从索引中搜索。
    @Test
    public void testMatchQuery() throws IOException {
        SearchRequest searchRequest = new SearchRequest("xc_course");
        searchRequest.types("doc");
        SearchSourceBuilder queryBuilder = new SearchSourceBuilder().query(QueryBuilders.matchQuery("description", "spring框架").operator(Operator.OR));
        queryBuilder.fetchSource(new String[]{"name", "studymodel"}, new String[]{});
        searchRequest.source(queryBuilder);
        SearchResponse searchResponse = highClient.search(searchRequest, RequestOptions.DEFAULT);
        printSearchResponse(searchResponse);
    }

    //使用minimum_should_match可以指定文档匹配词的占比
    //“spring开发框架”会被分为三个词：spring、开发、框架
    //设置"minimum_should_match": "80%"表示，三个词在文档的匹配占比为80%，即3*0.8=2.4，向上取整得2，表
    //示至少有两个词在文档中要匹配成功。
    @Test
    public void testMatchQuery2() throws IOException {
        SearchRequest searchRequest = new SearchRequest("xc_course");
        searchRequest.types("doc");
        SearchSourceBuilder queryBuilder = new SearchSourceBuilder()
                .query(QueryBuilders.matchQuery("description", "spring开发框架").minimumShouldMatch("80%"));
        queryBuilder.fetchSource(new String[]{"name", "studymodel"}, new String[]{});
        searchRequest.source(queryBuilder);
        SearchResponse searchResponse = highClient.search(searchRequest, RequestOptions.DEFAULT);
        printSearchResponse(searchResponse);
    }

    //multi Query 多字段匹配查询
    @Test
    public void testMultiQuery() throws IOException {
        SearchRequest searchRequest = new SearchRequest("xc_course").types("doc");
        SearchSourceBuilder queryBuilder = new SearchSourceBuilder()
                .query(QueryBuilders.multiMatchQuery("spring框架", "name", "description")
                        .minimumShouldMatch("50%")
                        .field("name", 10));
        queryBuilder.fetchSource(new String[]{"name", "description"}, new String[]{});
        searchRequest.source(queryBuilder);
        SearchResponse searchResponse = highClient.search(searchRequest, RequestOptions.DEFAULT);
        printSearchResponse(searchResponse);
    }

    //BoolanQuery 实现将多个查询组合起来
    @Test
    public void testBoolQuery() throws Exception {
        SearchRequest searchRequest = new SearchRequest("xc_course").types("doc");
        SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
        sourceBuilder.fetchSource(new String[]{"name", "description", "studymodel"}, new String[]{});
        //multiQuery
        MultiMatchQueryBuilder multiMatchQueryBuilder = QueryBuilders.multiMatchQuery("spring框架", "name", "description")
                .minimumShouldMatch("50%")
                .field("name", 10);
        //termQuery
        TermQueryBuilder termQueryBuilder = QueryBuilders.termQuery("studymodel", "201001");
        //boolQuery
        BoolQueryBuilder boolQuery = QueryBuilders.boolQuery();
        boolQuery.must(multiMatchQueryBuilder).must(termQueryBuilder);
        sourceBuilder.query(boolQuery);
        searchRequest.source(sourceBuilder);
        SearchResponse searchResponse = highClient.search(searchRequest, RequestOptions.DEFAULT);
        printSearchResponse(searchResponse);
    }

    //filter 过滤器，在bool查询中使用
    //sort 排序
    @Test
    public void testFilter() throws IOException {
        SearchRequest searchRequest = new SearchRequest("xc_course").types("doc");
        SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
        sourceBuilder.fetchSource(new String[]{"name", "description", "studymodel"}, new String[]{});
        //multiQuery
        MultiMatchQueryBuilder multiMatchQueryBuilder = QueryBuilders.multiMatchQuery("spring框架开发", "name", "description")
                .minimumShouldMatch("50%")
                .field("name", 10);
        //boolQuery
        BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery();
        boolQueryBuilder.must(multiMatchQueryBuilder);
        //filter
        boolQueryBuilder.filter(QueryBuilders.termQuery("studymodel", "201001"));
        boolQueryBuilder.filter(QueryBuilders.rangeQuery("price").gte(0).lte(100));
        sourceBuilder.query(boolQueryBuilder);
        //排序
        sourceBuilder.sort(new FieldSortBuilder("price").order(SortOrder.ASC));

        searchRequest.source(sourceBuilder);
        SearchResponse searchResponse = highClient.search(searchRequest, RequestOptions.DEFAULT);
        printSearchResponse(searchResponse);

    }

    //高亮显示
    @Test
    public void testHighlight() throws IOException {
        SearchRequest searchRequest = new SearchRequest("xc_course").types("doc");
        SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
        sourceBuilder.fetchSource(new String[]{"name", "description", "studymodel"}, new String[]{});
        //multiQuery
        MultiMatchQueryBuilder multiMatchQueryBuilder = QueryBuilders.multiMatchQuery("spring框架开发", "name", "description")
                .minimumShouldMatch("50%")
                .field("name", 10);
        BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery();
        boolQueryBuilder.must(multiMatchQueryBuilder);

        //高亮设置
        HighlightBuilder highlightBuilder = new HighlightBuilder();
        highlightBuilder.preTags("<tag>");
        highlightBuilder.postTags("</tag>");
        //设置高亮字段
        highlightBuilder.field(new HighlightBuilder.Field("name"));
        highlightBuilder.field(new HighlightBuilder.Field("description"));

        sourceBuilder.query(boolQueryBuilder).highlighter(highlightBuilder);
        searchRequest.source(sourceBuilder);
        SearchResponse searchResponse = highClient.search(searchRequest, RequestOptions.DEFAULT);

        SearchHits hits = searchResponse.getHits();
        for (SearchHit hit : hits) {
            Map<String, Object> sourceAsMap = hit.getSourceAsMap();
            String name = (String) sourceAsMap.get("name");
            //取出高亮字段内容
            Map<String, HighlightField> highlightFields = hit.getHighlightFields();
            if (highlightFields != null) {
                HighlightField nameField = highlightFields.get("name");
                if (nameField != null) {
                    Text[] fragments = nameField.getFragments();
                    StringBuffer stringBuffer = new StringBuffer();
                    for (Text str : fragments) {
                        stringBuffer.append(str.string());
                    }
                    name = stringBuffer.toString();
                }
            }
            String index = hit.getIndex();
            String type = hit.getType();
            String id = hit.getId();
            float score = hit.getScore();
            String sourceAsString = hit.getSourceAsString();
            String studymodel = (String) sourceAsMap.get("studymodel");
            String description = (String) sourceAsMap.get("description");
            System.err.println(name);
            System.err.println(studymodel);
            System.err.println(description);

        }

    }


    public void printSearchResponse(SearchResponse searchResponse) {
        SearchHits hits = searchResponse.getHits();
        SearchHit[] searchHits = hits.getHits();
        for (SearchHit hit : searchHits) {
            String index = hit.getIndex();
            String type = hit.getType();
            String id = hit.getId();
            float score = hit.getScore();
            String sourceAsString = hit.getSourceAsString();
            Map<String, Object> sourceAsMap = hit.getSourceAsMap();
            String name = (String) sourceAsMap.get("name");
            String studymodel = (String) sourceAsMap.get("studymodel");
            String description = (String) sourceAsMap.get("description");
            System.err.println(name);
            System.err.println(studymodel);
            System.err.println(description);
        }
    }

}
```

## DSL 聚合搜索

```java
import org.elasticsearch.search.aggregations.Aggregation;
import org.elasticsearch.search.aggregations.AggregationBuilders;
import org.elasticsearch.search.aggregations.bucket.histogram.DateHistogramInterval;
import org.elasticsearch.search.aggregations.bucket.histogram.Histogram;
import org.elasticsearch.search.aggregations.bucket.terms.ParsedStringTerms;
import org.elasticsearch.search.aggregations.bucket.terms.Terms;
import org.elasticsearch.search.aggregations.metrics.avg.Avg;

/**
* 聚合查询：
* （1）首先按照country国家来进行分组
* （2）然后在每个country分组内，再按照入职年限进行分组
* （3）最后计算每个分组内的平均薪资
*/
@Test
public void testSearch2() throws IOException {
    SearchRequest searchRequest = new SearchRequest("index2");
    SearchSourceBuilder queryBuilder = new SearchSourceBuilder();
    queryBuilder.aggregation(
            AggregationBuilders.terms("by_country").field("country")
                    .subAggregation(
                            AggregationBuilders.dateHistogram("by_year").field("join_date").dateHistogramInterval(DateHistogramInterval.YEAR)
                                    .subAggregation(AggregationBuilders.avg("avg_salary").field("salary"))
                    )
    );
    searchRequest.source(queryBuilder);

    SearchResponse response = restHighLevelClient.search(searchRequest, RequestOptions.DEFAULT);
    final Map<String, Aggregation> map = response.getAggregations().asMap();
    ParsedStringTerms by_country = (ParsedStringTerms) map.get("by_country");
    final Iterator<? extends Terms.Bucket> iterator = by_country.getBuckets().iterator();
    while (iterator.hasNext()) {
        //获取根据 country 的分组
        final Terms.Bucket country = iterator.next();
        System.err.println("country : " + country.getKey() + " = " + country.getDocCount());

        //获取根据 country 分组里的 根据 时间的分组
        Histogram by_year = (Histogram) country.getAggregations().asMap().get("by_year");
        final Iterator<? extends Histogram.Bucket> iter = by_year.getBuckets().iterator();
        while (iter.hasNext()) {
            final Histogram.Bucket year = iter.next();
            System.err.println("date : " + year.getKey() + " = " + year.getDocCount());

            //获取根据年分组的 平均 salary 的值
            Avg avg = (Avg) year.getAggregations().asMap().get("avg_salary");
            System.err.println("avg_salary = " + avg.getValue());
        }
    }

}
```

输出结果：

```text
country : china = 2
date : 2017-01-01T00:00:00.000Z = 2
avg_salary = 11000.0

country : usa = 2
date : 2015-01-01T00:00:00.000Z = 1
avg_salary = 23000.0
date : 2016-01-01T00:00:00.000Z = 1
avg_salary = 23000.0
```







