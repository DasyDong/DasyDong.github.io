---
title: "Django Project 性能优化"
layout: post
date: 2017-10-06 20:18
image: /img/images/markdown.jpg
headerImage: false
tag:
- performance
- python
category: blog
author: dong
description: Markdown summary with different options
# jemoji: '<img class="emoji" title=":ramen:" alt=":ramen:" src="https://assets.github.com/images/icons/emoji/unicode/1f35c.png" height="20" width="20" align="absmiddle">'
---

```
原文：A Guide to Performance Testing and Optimization With Python and Django
作者：IULIAN GULEA

翻译：雁惊寒

译文：http://www.iteye.com/news/32809
```

本文是在原文的基础上做代码的二次开发和文章的重新解读

### 目录
- [项目介绍](#demo)
- [性能分析](#analy)
- [ORM优化数据库查询](#select_related)
- [ORM仅提供相关的数据](#only)
- [编写自定义的序列化器](#serializer)
- [更新BaseHash包](#bascehash)
- [重构代码](#refactor)


<p>唐纳德·克努特（Donald Knuth）曾经说过："不成熟的优化方案是万恶之源。"然而，任何一个承受高负载的成熟项目都不可避免地需要进行优化。在本文中，我想谈谈优化Web项目代码的五种常用方法。虽然本文是以Django为例，但其他框架和语言的优化原则也是类似的。通过使用这些优化方法，文中例程的查询响应时间从原来的77秒减少到了3.7秒。</p>
<p>本文用到的例程是从一个我曾经使用过的真实项目改编而来的，是性能优化技巧的典范。如果你想自己尝试着进行优化，可以在GitHub上获取优化前的初始代码，并跟着下文做相应的修改。我使用的是Python 2，因为一些第三方软件包还不支持Python 3</p>

<h2 id="demo">示例代码介绍</h2>
[django_optimise_demo](https://github.com/DasyDong/django_optimise_demo) 这个Web项目只是简单地跟踪每个地区的房产价格。因此，只有两种模型：
```python
# houses/models.py
from utils.hash import Hasher
class HashableModel(models.Model):

    """Provide a hash property for models."""
    class Meta:
        abstract = True

    @property
    def hash(self):
        return Hasher.from_model(self)

class Country(HashableModel):

    """Represent a country in which the house is positioned."""

    name = models.CharField(max_length=30)
    def __unicode__(self):
        return self.name

class House(HashableModel):

    """Represent a house with its characteristics."""

    # Relations

    country = models.ForeignKey(Country, related_name='houses')

    # Attributes
    address = models.CharField(max_length=255)
    sq_meters = models.PositiveIntegerField()
    kitchen_sq_meters = models.PositiveSmallIntegerField()
    nr_bedrooms = models.PositiveSmallIntegerField()
    nr_bathrooms = models.PositiveSmallIntegerField()
    nr_floors = models.PositiveSmallIntegerField(default=1)
    year_built = models.PositiveIntegerField(null=True, blank=True)
    house_color_outside = models.CharField(max_length=20)
    distance_to_nearest_kindergarten = models.PositiveIntegerField(null=True, blank=True)
    distance_to_nearest_school = models.PositiveIntegerField(null=True, blank=True)
    distance_to_nearest_hospital = models.PositiveIntegerField(null=True, blank=True)
    has_cellar = models.BooleanField(default=False)
    has_pool = models.BooleanField(default=False)
    has_garage = models.BooleanField(default=False)
    price = models.PositiveIntegerField()
    def __unicode__(self):
        return '{} {}'.format(self.country, self.address)
```

<p>抽象类 HashableModel提供了一个继承自模型并包含 hash属性的模型，这个属性包含了实例的主键和模型的内容类型。 这能够隐藏像实例ID这样的敏感数据，而用散列进行代替。如果项目中有多个模型，而且需要在一个集中的地方对模型进行解码并要对不同类的不同模型实例进行处理时，这可能会非常有用。 请注意，对于本文的这个小项目，即使不用散列也照样可以处理，但使用散列有助于展示一些优化技巧。

这是 Hasher类：</p>
```python
# utils/hash.py
import basehash

class Hasher(object):

    @classmethod
    def from_model(cls, obj, klass=None):
        if obj.pk is None:
            return None
        return cls.make_hash(obj.pk, klass if klass is not None else obj)

    @classmethod
    def make_hash(cls, object_pk, klass):
        base36 = basehash.base36()
        content_type = ContentType.objects.get_for_model(klass, for_concrete_model=False)
        return base36.hash('%(contenttype_pk)03d%(object_pk)06d' % {
            'contenttype_pk': content_type.pk,
            'object_pk': object_pk
        })

    @classmethod
    def parse_hash(cls, obj_hash):
        base36 = basehash.base36()
        unhashed = '%09d' % base36.unhash(obj_hash)
        contenttype_pk = int(unhashed[:-6])
        object_pk = int(unhashed[-6:])
        return contenttype_pk, object_pk

    @classmethod
    def to_object_pk(cls, obj_hash):
        return cls.parse_hash(obj_hash)[1]

```
由于我们想通过API来提供这些数据，所以我们安装了Django REST框架并定义以下序列化器和视图：
```python
# houses/serializers.py

class HouseSerializer(serializers.ModelSerializer):

    """Serialize a `houses.House` instance."""

    id = serializers.ReadOnlyField(source="hash")
    country = serializers.ReadOnlyField(source="country.hash")
    class Meta:
        model = House
        fields = (
            'id',
            'address',
            'country',
            'sq_meters',
            'price'
        )

# houses/views.py
class HouseListAPIView(ListAPIView):
    model = House
    serializer_class = HouseSerializer
    country = None

    def get_queryset(self):
        country = get_object_or_404(Country, pk=self.country)
        queryset = self.model.objects.filter(country=country)
        return queryset

    def list(self, request, *args, **kwargs):
        # Skipping validation code for brevity
        country = self.request.GET.get("country")
        self.country = Hasher.to_object_pk(country)
        queryset = self.get_queryset()
        serializer = self.serializer_class(queryset, many=True)
        return Response(serializer.data)
```
现在，我们将用一些数据来填充数据库（使用 factory-boy生成10万个房屋的实例：一个地区5万个，另一个4万个，第三个5千个），并准备测试应用程序的性能。

```python
from factory.declarations import SubFactory
from factory.django import DjangoModelFactory
from factory.faker import Faker
from factory.fuzzy import FuzzyInteger

from houses.models import House, Country


class CountryFactory(DjangoModelFactory):
    class Meta:
        model = Country

    name = Faker('country')


class HouseFactory(DjangoModelFactory):
    class Meta:
        model = House

    country = SubFactory(CountryFactory)

    address = Faker('address')
    sq_meters = FuzzyInteger(30, 400)
    kitchen_sq_meters = FuzzyInteger(5, 100)
    nr_bedrooms = FuzzyInteger(1, 7)
    nr_bathrooms = FuzzyInteger(1, 4)
    nr_floors = FuzzyInteger(1, 4)
    year_built = FuzzyInteger(1960, 2017)
    house_color_outside = Faker('safe_color_name')
    distance_to_nearest_kindergarten = FuzzyInteger(100, 5000)
    distance_to_nearest_school = FuzzyInteger(100, 5000)
    distance_to_nearest_hospital = FuzzyInteger(100, 5000)
    has_cellar = FuzzyInteger(0, 1)
    has_pool = FuzzyInteger(0, 1)
    has_garage = FuzzyInteger(0, 1)
    price = FuzzyInteger(50000, 500000)

```
<h2 id="analy">性能分析</h2>
性能优化其实就是测量
在一个项目中我们需要测量下面这几个方面：

执行时间

代码的行数

函数调用次数

分配的内存

其他

但是，并不是所有这些都要用来度量项目的执行情况。一般来说，有两个指标比较重要：执行多长时间、需要多少内存。

在Web项目中，响应时间（服务器接收由某个用户的操作产生的请求，处理该请求并返回结果所需的总的时间）通常是最重要的指标，因为过长的响应时间会让用户厌倦等待，并切换到浏览器中的另一个选项卡页面。

在编程中，分析项目的性能被称为profiling。为了分析API的性能，我们将使用Silk包。在安装完这个包，并调用 /api/v1/houses/?country=3，可以得到如下的结果：

<img src='/img/images/django-optimise/sql_origin.png'>

<h2 id="select_related">ORM优化数据库查询</h2>
这个问题的根源是，Django中的查询是惰性的。这意味着在你真正需要获取数据之前它不会访问数据库。同时，它只获取你指定的数据，如果需要其他附加数据，则要另外发出请求。

这正是本例程所遇到的情况。当通过 House.objects.filter(country=country)来获得查询集时，Django将获取特定地区的所有房屋。但是，在序列化一个 house实例时， HouseSerializer需要房子的 country实例来计算序列化器的 country字段。由于地区数据不在查询集中，所以django需要提出额外的请求来获取这些数据。对于查询集中的每一个房子都是如此，因此，总共是五万次。

当然，解决方案非常简单。为了提取所有需要的序列化数据，你可以在查询集上使用 select_related()。因此， get_queryset函数将如下所示：

```python
def get_queryset(self):

    country = get_object_or_404(Country, pk=self.country)

    queryset = self.model.objects.filter(country=country).select_related('country')

    return queryset
```
我们来看看这对性能有何影响：
<img src='/img/images/django-optimise/sql_related.png'>

<h2 id="only">ORM仅提供相关的数据</h2>
默认情况下，Django会从数据库中提取所有字段。但是，当表有很多列很多行的时候，告诉Django提取哪些特定的字段就非常有意义了，这样就不会花时间去获取根本用不到的信息。在本案例中，我们只需要5个字段来进行序列化，虽然表中有17个字段。明确指定从数据库中提取哪些字段是很有意义的，可以进一步缩短响应时间。

Django可以使用 defer()和 only()这两个查询方法来实现这一点。第一个用于指定哪些字段不要加载，第二个用于指定只加载哪些字段。
```python
def get_queryset(self):

    country = get_object_or_404(Country, pk=self.country)

    queryset = self.model.objects.filter(country=country)

        .select_related('country')

        .only('id', 'address', 'country', 'sq_meters', 'price')

    return queryset
```
我们看到sql查询耗时下降， 但这里api时间没太多变化，我估计是和数据量不大有关

<img src='/img/images/django-optimise/sql-only.png'>


<h2 id ="serializer">编写自定义的序列化器</h2>
以上数据库的优化，已经将时间大大优化，接下来我们进步一深入分析下代码。因为我们用silk时， 选择了生成prof文件，
所以本文将使用 [snakeviz](http://jiffyclub.github.io/snakeviz/)
```
snakeviz profiles/acc120c6-29f1-44aa-8ced-a9e45fb55750.prof
```

这是上文一个请求的二进制分析文件的可视化图表：
<img src='/img/images/django-optimise/basehash.png'>

我们发现prime文件和serializer文件花销大，prime文件时basehash包的代码，
serializer时django序列化代码，所以我们考虑先自定义一个序列化

<p style="color:rgb(21, 153, 87)">原文中这样提到：

"现在看起来好多了，由于没有使用DRF序列化代码，所以响应时间几乎减少了一半。

另外还有一个结果：在请求/响应周期内完成的总的函数调用次数从15,859,427次（上面1.2节的请求次数）减少到了9,257,469次。这意味着大约有三分之一的函数调用都是由Django REST Framework产生的。"
</p>
注： 亲身体验了下， 实际上drf现在的效率已经很高，此优化没有明显改变

```python
class HousePlainSerializer(object):
    """
    Serializes a House queryset consisting of dicts with
    the following keys: 'id', 'address', 'country',
    'sq_meters', 'price'.
    """
    @staticmethod
    def serialize_data(queryset):
        """
        Return a list of hashed objects from the given queryset.
        """
        return [

            {
                'id': entry.hash,
                'address': entry.address,
                'country': entry.country.hash,
                'sq_meters': entry.sq_meters,
                'price': entry.price
            } for entry in queryset
        ]


class HouseListAPIView(ListAPIView):
    model = House
    serializer_class = HouseSerializer
    plain_serializer_class = HousePlainSerializer  # <-- added custom serializer
    country = None

    def get_queryset(self):

        if self.country:
            country = get_object_or_404(Country, pk=self.country)
            queryset = self.model.objects.filter(country=country)\
            .select_related('country')\
            .only('id', 'address', 'country', 'sq_meters', 'price')
        else:
            queryset = self.model.objects.all().select_related('country')\
            .only('id', 'address', 'country', 'sq_meters', 'price')[:self.count]
        return queryset

    def list(self, request, *args, **kwargs):
        # Validation code to check for `country` param should be here
        self.country = self.request.GET.get("country")
        self.count = self.request.GET.get("count") or 100
        if self.country:
            self.country = Hasher.to_object_pk(self.country)
        queryset = self.get_queryset()
        data = self.plain_serializer_class.serialize_data(queryset)
        return Response(data)
```


<h2 id ="basehash">更新第三方包</h2>
目前，这是代码的主要性能瓶颈，但同时，这不是我们自己写的代码，而是用的第三方包。

在这种情况下，我们可以做的事情将非常有限：

1. 检查包的最新版本（希望能有更好的性能）。

2. 寻找另一个能够满足我们需求的软件包。

3. 我们自己写代码，并且性能优于目前使用的软件包。

幸运的是，我们找到了一个更新版本的 basehash包。原代码使用的是v.2.1.0，而新的是v.3.0.4。

当查看v.3的发行说明时，这一句话看起来令人充满希望：
```
"使用素数算法进行大规模的优化。"
```
<img src='/img/images/django-optimise/upgrade.png'>

注：升级新版版后，unhash需要做下小更改，可以查看重构后的源码

<h2 id="refactor">重构代码</h2>
我们知道basehash初始化操作是耗时的， 我们的代码里有两处调用，所以可以考虑把代码提到方法外面，做类的属性
```python
class Hasher(object):
    base36 = basehash.base36()  # <-- initialize hasher only once
```

<h2>结论</h2>
最后api耗时2s返回，所以尽可能的做优化是很重要的
