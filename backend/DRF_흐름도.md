# 흐름도

<aside>
⚒️  Model   →   Viewset   →   query       
 →   serializer   →   Viewset   →   Response  
</aside>

즉,
Model => [middleware] => Viewset => [only] => query => [values] => serializer => viewset => response이다.  


1. **Model (DB)이 존재한다**
    
    ### ***`middleware`***
    
    : 각종 미들웨어등을 이 단계에서 거친후 viewset으로 간다
    
2. **Viewset을 통해 query를 먼저 부른다 (queryset  부분)**
    - 대부분 `object.all()` 로 부르는데 정말 대용량아니면 속도차이도 딕셔너리와 크게 차이가없고, 객체로 받아오기때문에 좀더 활용이 가능하기때문이다
    - `queryset = Model.objects.all()` : object<queryset> 타입으로 불러온다
    - `queryset = Model.objects.values()` : dict 타입으로 불러온다
    
    ### ***`.only`***
    
    **장점 : 값을 전부 조회한후 커스텀하는게 아니라, 처음 조회하는 부분에서부터 커스텀할 수 있다. 따라서 처음에 가지고 오는 쿼리를 줄일수 있어서 성능면에서 좋다
    
    **주의점 : only에서 정의하지않은 항목을 serialize에서 요청한다면 그만큼의 쿼리를 계속 날리게되어 오히려 성능이 떨어진다.항상 .only와 serialize의 항목을 맞춰줘야한다.
    
3. **Viewset을 통해 Serialize를 부른다 (serialize_class 부분)**
    
    ### *`.values`*
    
    values 로 qs를 요청할경우, 이 단계에서 거쳐서 serialize로 넘어간다. 쿼리를 전부 부른후, 커스텀하기 때문에 필요에 따라서 only 를 고려해야한다.
    
4. **Serialize에서 틀을 보여주는 작업을 거친다**
    - model자체는 건드릴수없는 서드파티개념이기때문에, values 커스텀과 비슷하게 될 수 밖에 없다
5. **Viewset으로 Serialize작업을 거친내용이 다시 들어온다**
6. **Response 값으로, 지정된 Router에 맞게 반환된다** (Mixins를 쓸경우 isValid(), json작업이 포함되어있다)
