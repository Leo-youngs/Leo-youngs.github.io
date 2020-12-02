---
title: django-full-text-search
date: 2020-10-27 16:15:04
tags: 全文检索
categories: 后端

---


## 一. mysql 的全文索引

从MySQL 5.7开始，MySQL内置了ngram全文检索插件，用来支持中文分词，并且对MyISAM和InnoDB引擎有效。

1. 必要的参数设置

    ```bash
    # 在 /etc/mysql/mysql.conf.d/mysqld.cnf 中添加分词以及最小词语长度
    ft_min_word_len = 2
    ngram_token_size = 2

    echo 'ft_min_word_len = 2
    ngram_token_size = 2' >> mysqld.cnf

    /etc/init.d/mysql restart

    # 查看配置
    SHOW VARIABLES LIKE 'ft_min_word_len';

    SHOW VARIABLES LIKE 'ngram_token_size';
    ```

2. mysql 索引配置

    ```sql
    -- CREATE FULLTEXT INDEX knowledge_knowledge_content_index ON knowledge_knowledge ( content, title );

    -- 这个方式创建生效成功
    ALTER TABLE knowledge_knowledge ADD FULLTEXT INDEX knowledge_knowledge_content_index ( content, title ) WITH PARSER ngram;
    ```

3. django 中适用配置

    ```python
    sql = "SELECT * from knowledge_knowledge where match(content, title) against('原件' in BOOLEAN MODE);"

     k = Knowledge.objects.raw(sql)
    ```

优点: 不需要引入过多的插件，直接利用数据库的功能。
缺点: 随着数据量的增加性能可能成为主要瓶颈，而且不利于项目的扩展

## 二. drf_haystack whoosh jieba

文档参考

- [drf_haystack](https://drf-haystack.readthedocs.io/en/latest/index.html)

1. 项目依赖下载

    ```bash
    pip install django-haystack
    pip install drf_haystack
    pip install Whoosh
    pip install jieba

    # 解决 ImportError: cannot import name connections

    pip uninstall haystack
    pip uninstall django-haystack
    pip install django-haystack
    ```

2. 基本配置

    - 修改settings 文件

    ```python
    # 1. 修改settings 文件

    # INSTALLED_APPS 注意放在最前面

    INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'haystack',
    'rest_framework',
    'knowledge'
    ]

    HAYSTACK_CONNECTIONS = {
    'default': {
        # 'ENGINE': 'haystack.backends.whoosh_backend.WhooshEngine',
        'ENGINE': 'main.whoosh_cn_backend.WhooshEngine',
        'PATH': os.path.join(os.path.dirname(__file__), 'whoosh_index'),
        },
    }

    # 当添加、修改、删除数据时，自动生成索引
    HAYSTACK_SIGNAL_PROCESSOR = 'haystack.signals.RealtimeSignalProcessor'
    # 设置每页显示的结果数量
    HAYSTACK_SEARCH_RESULTS_PER_PAGE = 10
    ```

    - 添加索引配置文件，项目中新建 search_indexes.py

    ```python
    from knowledge.models.Knowledge import Knowledge
    from haystack import indexes


    class KnowledgeIndex(indexes.SearchIndex, indexes.Indexable):

    text = indexes.CharField(document=True, use_template=True)
    title = indexes.CharField(model_attr="title")
    content = indexes.CharField(model_attr="content")
    tag = indexes.CharField(model_attr="tag")
    creator = indexes.CharField(model_attr="creator")
    id = indexes.CharField(model_attr="pk")
    autocomplete = indexes.EdgeNgramField()

    @staticmethod
    def prepare_autocomplete(obj):
        return " ".join((
        obj.title,
        ))

    def get_model(self):
        return Knowledge

    def index_queryset(self, using=None):
        return self.get_model().objects.all()

    ```

    - 结巴分词替换whoosh 默认的分词

    ```python
    # 1. 拷贝 `haystack/backends/whoosh_backends.py` 到当前目录

    # 2. 搜索 并修改
    schema_fields[field_class.index_fieldname] = TEXT(stored=True, analyzer=StemmingAnalyzer(), field_boost=field_class.boost, sortable=True)

    from jieba.analyse import ChineseAnalyzer

    ...
    #注意先找到这个再修改，而不是直接添加  
    schema_fields[field_class.index_fieldname] = TEXT(stored=True, analyzer=ChineseAnalyzer(),field_boost=field_class.boost, sortable=True)

    ```

    - 配置 路由 视图 序列化

    ```python
    # 1. urls.py
    from rest_framework import routers

    router = routers.SimpleRouter(trailing_slash=False)

    router.register("location/search", KnowledgeSearchViewSet, base_name="location-search")

    router.register("search", viewset=SearchViewSet, base_name="search")  # MLT name will be 'search-more-like-this'.

    # 2. serializers.py
    from drf_haystack.serializers import HaystackSerializer
    from knowledge.search_indexes import KnowledgeIndex
    from drf_haystack.serializers import HaystackSerializer


    class KnowledgeSearchSerializer(HaystackSerializer):
        # more_like_this = serializers.HyperlinkedIdentityField(view_name="search-more-like-this", read_only=True)

        class Meta:
            index_classes = [KnowledgeIndex]
            fields = ['title', 'tag', 'content', 'creator', 'id', 'autocomplete']
            ignore_fields = ["autocomplete"]

    class AutocompleteSerializer(HaystackSerializer):

        class Meta:
            index_classes = [KnowledgeIndex]
            fields = ["address", "city", "zip_code", "autocomplete"]
            ignore_fields = ["autocomplete"]

            # The `field_aliases` attribute can be used in order to alias a
            # query parameter to a field attribute. In this case a query like
            # /search/?q=oslo would alias the `q` parameter to the `autocomplete`
            # field on the index.
            field_aliases = {
                "q": "autocomplete"
            }


    class SearchSerializer(HaystackSerializer):

        more_like_this = serializers.HyperlinkedIdentityField(view_name="search-more-like-this", read_only=True)

        class Meta:
            index_classes = [KnowledgeIndex]
            fields = ['title', 'tag', 'content', 'creator', 'id']
    # 3. views.py

    from drf_haystack.viewsets import HaystackViewSet

    from knowledge.serializers.KnowledgeSerializer import KnowledgeSearchSerializer, AutocompleteSerializer, SearchSerializer
    from drf_haystack.filters import HaystackAutocompleteFilter
    from drf_haystack.viewsets import HaystackViewSet
    from drf_haystack.mixins import MoreLikeThisMixin
    from knowledge.models.Knowledge import Knowledge

    # ViewSet
    class KnowledgeSearchViewSet(HaystackViewSet):
        index_models = [Knowledge]
        serializer_class = KnowledgeSearchSerializer

    class AutocompleteSearchViewSet(HaystackViewSet):

        index_models = [Knowledge]
        serializer_class = AutocompleteSerializer
        filter_backends = [HaystackAutocompleteFilter]

    class SearchViewSet(MoreLikeThisMixin, HaystackViewSet):
        index_models = [Knowledge]
        serializer_class = SearchSerializer
    ```

    - 生成索引

    ```bash
    python3 manage.py rebuild_index
    ```
