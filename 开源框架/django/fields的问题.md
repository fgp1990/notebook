请求中 feilds=\*\*\*\*，这样就只返回对应字段的值。这个东西在原生的 django 或者 drf 中是没有的，需要自己改对应的方法。
下面这里这个 fields 仅仅是控制返回哪些字段，并不能指定字段。就是这里设置好了，要返回就一起返回，返回这里设置过的。

```python
class CkhTableSerializer(serializers.ModelSerializer):
    class Meta:
        model = CkhTable
        fields = "__all__"
```

如果要实现上述功能，需要修改 serializers.ModelSerializer 这个类

```python
class DynamicFieldsModelSerializer(serializers.ModelSerializer):
    """
    A ModelSerializer that takes an additional `fields` argument that
    controls which fields should be displayed.
    """

    def __init__(self, *args, **kwargs):
        # 根据初始化参数筛选fields
        fields = kwargs.pop('fields', None)
        exclude = kwargs.pop("exclude", None)
        depth = kwargs.pop("depth", None)
        super(DynamicFieldsModelSerializer, self).__init__(*args, **kwargs)

        #  实例化传递参数fields
        self.param_fields = []
        if fields is not None:
            # Drop any fields that are not specified in the `fields` argument.
            allowed = set(fields)
            existing = set(self.fields)
            # for field_name in existing - allowed:
            #     self.fields.pop(field_name)
            for field_name in existing:
                if field_name not in allowed:
                    self.fields.pop(field_name)
                else:
                    self.param_fields.append(field_name)

        if depth is not None:
            try:
                depth = int(depth)
                if depth > 0 or depth < 5:
                    self.Meta.depth = depth
            except ValueError:
                pass

        # 根据query param 筛选fields
        self.query_fields = []
        if 'request' in self.context and self.context.get('request'):
            query_fields = self.context['request'].query_params.get('fields')
            if query_fields:
                query_fields = query_fields.split(',')
                allowed = set(query_fields)
                existing = set(self.fields.keys())
                for field_name in existing - allowed:
                    self.fields.pop(field_name)
                self.query_fields = list(allowed)

            # 根据query param  设置depth
            # 框架实现一个内部serializer, 不是使用自己定义的serializer, 字段不可控, 暂时不用, 以后解决..
            # try:
            #     depth = int(self.context['request'].query_params.get('depth', 0))
            #     if depth > 0 or depth < 10:
            #         self.Meta.depth = depth
            # except ValueError:
            #     pass

        # 排除 exclude 字段
        self.exclude_fields = []
        if exclude is not None:
            for field_name in exclude:
                self.exclude_fields.append(field_name)
                self.fields.pop(field_name)

    def check_query_field(self, key):
        # 在排除字段中
        if key in self.exclude_fields:
            return False
        # 如果没有query field, 返回True
        if not self.query_fields and not self.param_fields:
            return True
        return key in self.query_fields or key in self.param_fields
```
