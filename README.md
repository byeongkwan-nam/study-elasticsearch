# 4. Managing Documents

Created: Sep 02, 2019 9:14 PM
Updated: Sep 02, 2019 10:27 PM

# 문서 추가하기

문서를 추가하기 위해서는 두 가지를 정해야 합니다. 

1) 어떤 방법으로  ⇒ method (POST, GET, PUT, DELETE)

2) 어디에?          ⇒ index 명

product 라는 이름의 인덱스를 만들고, type명을 default 라고 했을 때 쿼리는 다음과 같이 구성됩니다.

    POST /product/default

하지만 위 코드에는 하나가 빠져 있습니다. 바로 3) 무엇을? 입니다. 다시 말해 어떤 문서를 추가할 것인지 상세히 써주어야 하고, elasticsearch에서는 이 문서를 JSON을 이용해 표현합니다. 넣고 싶은 임의의 JSON이 바로 우리의 문서가 됩니다. 문서를 추가하는 완전한 쿼리는 다음과 같습니다.

    POST /product/default
    {
    	"title": "Introduction to Elasticsearch",
    	"instructor": {
    		"firstName": "Bk",
    		"lastName": "Nam"
    	}
    }

Bk Nam이라는 강사가 가르치는 엘라스틱서치 기본 강의를 만들었습니다. 이 쿼리를 실제로 날려보면 어떤 모습일까요? 다음과 같습니다. 

    {
      "_index": "product",
      "_type": "default",
      "_id": "eczw8WwBvHIvKPgtDUgl",
      "_version": 1,
      "result": "created",
      "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
      },
      "_seq_no": 211,
      "_primary_term": 3
    }

보시면 하나 흥미로운 점을 찾으실 수 있습니다. 바로 _id 인데요. 저는 아무런 키도 주지 않았는데 알아서 id를 무작위의 20자리 문자열을 배당한 걸 알 수 있습니다. 우리는 그리고 아이디를 지정할 수 있습니다. 바로 다음처럼요.

    POST /product/default/10
    {
    	"title": "Introduction to Elasticsearch",
    	"instructor": {
    		"firstName": "Bk",
    		"lastName": "Nam"
    	}
    }

그럼 다음과 같은 결과가 생깁니다.

    {
      "_index": "product",
      "_type": "default",
      "_id": "1",
      "_version": 1,
      "result": "created",
      "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
      },
      "_seq_no": 202,
      "_primary_term": 3
    }

그런데 하나 이상한 점 느끼지 못하셨나요? 지금 문서를 생성할 때, 흐름을 잘 보면 우리는 type의 이름을 정하지 않고 default라고 임의로 정했는데, ES가 알아서 잘 처리해줬습니다. 잘 생각해보면 신기한 일인데요. 사실 ES는 임의의 type을 사용자가 지정할 경우 request를 받았을 때 자동으로 type을 생성합니다.

그리고 이는 인덱스에도 적용이 됩니다. 만약 우리가 product라는 인덱스를 만들지 않고 바로 생성 쿼리부터 날렸다고 하더라도, ES는 알아서 인덱스와 타입을 만들고 그 안에 JSON 에 맞는 문서를 생성해줍니다. 그리고 이러한 기능들은 원할 경우 작동하지 않게 할 수 있습니다.

# 문서 검색하기

저장한 문서를 검색하는 방법은 여러 가지가 있겠습니다. 이번에는 문서의 아이디를 안다고 가정할 때, 해당 문서를 가져오는 방법에 대해 알아보겠습니다. 쿼리와 결과는 다음과 같습니다.

    GET /product/default/1

    {
      "_index": "product",
      "_type": "default",
      "_id": "1",
      "_version": 3,
      "found": true,
      "_source": {
        "title": "Introduction to Elasticsearch",
        "instructor": {
          "firstName": "Bk",
          "lastName": "Nam"
        }
      }
    }

말 그대로 아이디가 1인 문서를 가져왔습니다. 원하던 대로 처음에 넣었던 문서가 맞는데.. 보다 보니 눈에 띄는 필드가 하나 있습니다. "_version"? 그리고 3? 버전이라 함은 업데이트가 얼마나 이루어졌느냐 - 이걸 나타내는 용도로 많이 쓰잖아요? 그러면 버전이 1이 나와야 할 거 같은데 3이 나왔습니다. 이게 대체 어떻게 된 일일까요?

# 문서 대체(replace)하기

결론부터 이야기하자면 사실 제가 아이디 1번인 문서를 여러분 모르게 두 번 업데이트한 것입니다. 뭐라구요? 네 맞습니다. 여러분이 글을 읽는 사이에 제가 몰래 문서를 두 번 업데이트했고, 그 결과 버전 필드가 3이 나오게 된 것입니다. 제가 갑자기 욕심쟁이가 돼서 무료 강좌에 가격을 붙이고, 그 내용을 인덱스에 반영하고 싶었던 것이죠. 그러면 이 내용은 어떻게 이루어졌을까요?

    PUT /product/default/1
    {
    	"title": "Introduction to Elasticsearch",
    	"instructor": {
    		"firstName": "Bk",
    		"lastName": "Nam"
    	},
    	"price": 200000
    }

결과

    {
      "_index": "product",
      "_type": "default",
      "_id": "1",
      "_version": 4,
      "result": "updated",
      "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
      },
      "_seq_no": 204,
      "_primary_term": 3
    }

욕심쟁이처럼 가격을 한 번 더 올렸더니 이번엔 버전이 4가 되었습니다. 문서를 변경한 것이지요. 하지만 이번에도 불편한 점이 하나 있습니다. 가격 하나 바꾸자고 강의 제목부터 강의자까지 다 입력해야 할까요? 그러다가 오타라도 나면요? 내가 원하는 필드만 추가하거나, 변경하는 방법은 없을까요? 

# 문서 수정(update)하기

    POST /product/default/1/_update
    {
    	"doc": {
    		"price": 190000,
    		"tags": [ "Elasticsearch" ]
    	}
    }

    {
      "_index": "product",
      "_type": "default",
      "_id": "1",
      "_version": 5,
      "result": "updated",
      "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
      },
      "_seq_no": 205,
      "_primary_term": 3
    }
    

1) 어떻게? POST, _update

2) 어디에? 1번 문서에

3) 무엇을? "doc" 안에 있는 내용들을.

위와 같이 해주면 보시는 바와 같이 버전이 5로 바뀌고, result 필드는 updated로 바뀌어 있습니다. 쉽죠? 쉽지만, 알고 보면 그 안에서 일어나는 일은 기대와 사뭇 다릅니다. 무엇이 다를까요? 바로.. 엘라스틱서치에서 문서는 변경 불가능(immutable)합니다. 네? 하지만 방금 분명히 문서가 수정되는 것을 보지 않았나요? 그러면 수정가능(mutable)한 것 같지만... 사실은 아닙니다.

그럼 구체적으로 어떤 일이 일어나는 것일까요? 그리고 왜 엘라스틱서치의 문서(데이터)는 수정불가능 할까요? 사실 업데이트를 요청했을 때, ES는 (1) 해당 문서를 획득하고, (2) 수정 요구에 맞추어 새로운 문서를 만든 다음, (3) 새로운 문서로 과거 문서를 대체합니다. 다시 말해 기존 데이터(A)와 새로운 데이터(A')는 하나의 인덱스 안에서 공존하고 있습니다. 그리고 이 과정은 자동으로 일어나기 때문에, 우리가 굳이 여러 번 request를 날릴 필요가 없고, 따라서 네트워크 비용이 발생하지 않습니다. 

# 스크립트로 업데이트하기

만약 제가 갑자기 기분이 너무 좋아져서 강좌의 가격을 20만 원에서 천 원으로 대폭 할인한다고 가정해보겠습니다. 지금까지 배운 방법으로는 다음과 같은 단계를 거쳐야 합니다. 

(1) GET /product/default/1을 통해서 현재 가격을 획득하고,

(2) 19만 9천 원을 빼준 다음

(3) POST /product/default/1/_update 쿼리를 통해 가격을 1천 원으로 바꿔줍니다.

하지만 이 방법 외에도 스크립트를 통해 한 번에 가격을 바꿀 수 있습니다. 바로 아래와 같은 쿼리로 가능합니다. 특기할 만한 점이라면 이번에는 "doc" property가 아니라 "script" property를 사용합니다.

    POST /product/default/1/_update
    {
    	"script": "ctx._source.price -= 199000"
    }

스크립트는 이것 뿐만 아니라 아주 다양한 기능을 가지고 있는데, 이번 글에서는 우선 이 정도로만 다루고 넘어가고 다음 기회에 조금 더 상세히 써보도록 하겠습니다. 지금은 기초 기능을 빠르게 훑는게 목적이니까요.

# Upsert: 있으면 업데이트하고, 없으면 생성하기

... 안되는데? 이거는 ..ㅎㅋㅋㅋ 문법이 좀 다른 듯/

# 문서 삭제하기

이쯤 되면 어떤 방식으로 삭제할지는 감이 오시지요?

    DELETE /product/default/1

이렇게 하면 됩니다! 그러면 하나 질문을 던져보겠습니다. 한 번에 여러 문서를 삭제하려면 어떻게 하면 될까요? 예시를 들기 위해 우선 두 문서를 추가해보겠습니다.

    POST /product/default
    {
      "name": "Data processing",
      "category": "course"
    }
    
    POST /product/default
    {
      "name": "The art of war",
      "category": "book"
    }

이제 두 문서 중에서, 카테고리가 책인 문서들을 한 번에 삭제하고 싶습니다. 어떻게 하면 될까요? 우리는 Delete by query 라는 API를 사용하면 됩니다.

어떻게 구성하면 될까요? 생각해보니 다음 요소가 필요하겠습니다.

(1) 어떤 인덱스를 대상으로? ⇒ /product/default

(2) 어떤 방법으로? ⇒ POST & _delete_by_query

음 그리고 인덱스에게 명령을 내릴 겁니다. 카테고리가 정확히 책과 일치하는 녀석들을 삭제하라구요. 다시 써보겠습니다. 명령(query)을 내릴 겁니다. 카테고리(category)가 정확히 책(book)과 일치(match)하는 녀석들을 삭제하라구요. 이걸 순서에 맞게 써보면 다음과 같습니다.

    POST /product/default/_delete_by_query
    {
      "query": {
        "match": {
          "category": "book"
        }
      }
    }

처음 보는 거라 낯설 수 있는데, 어렵게 생각하지 말고 다시 찬찬히 생각해봅시다. 잘 생각해보면 의도하는 바를 명확히 드러내는 명령문(query)임을 알 수 있습니다. 어순이 한국어와 조금 다르긴 한데, 약간의 억지(?)를 부려보면 ㅋㅋ "카테고리(category)가 정확히 책(book)과 일치(match)하는 녀석들을 삭제해라"와 비슷하지 않나요?






# 5. Mapping

# Schema == Mapping

RDBMS에서 데이터를 삽입하려면 반드시 사전에 테이블의 스키마를 정의해야 했습니다. 마찬가지로 ES의 index에 document, 문서를 삽입하려면 해당 문서가 어떻게 생겨먹었는지 먼저 정의해주어야 합니다. 이 과정을 Mapping이라고 부릅니다.

여태까지 했던 것처럼 간단한 사용을 위해서는 사실 매핑을 정의하지 않아도 됩니다. 엘라스틱서치가 알아서 해주니까요. 하지만 조금 더 통제권을 가지고 제대로 갖고 놀기 위해서는 어떻게 매핑을 정의하고, 어떻게 사용해야 하는지 알 필요가 있습니다. 그런데 하나 궁금한 점이 생기네요. 어떻게 ES는 미리 매핑을 정의하지 않아도 문서를 삽입할 수 있도록 했을까요?

# Dynamic Mapping

미리 매핑을 명시적으로 정의할 수도 있지만, 여태까지 해왔던 것처럼 다짜고짜 데이터를 집어넣어도 ES는 찰떡처럼 알아듣고 적절한 자료형을 찾아줍니다. 이를 다이나믹 매핑이라고 부르는데요. 많이 쓰이는 특정 데이터 형태들 - date, boolean, text, keyword ... - 에 대해서는 ES가 자동으로 설정할 수 있게끔 만들어져 있을 뿐더러 이를 위해 우리가 해야할 일은 아무 것도 없습니다. 구체적인 내용에 대해서는 다음 기회에 알아보는 걸로 하고, 이번에는 다이나믹 매핑을 확인할 수 있는 쿼리를 실행해보도록 하겠습니다.

    GET /test/default/1
    {
      "created" : "2019/09/03",
      "isExist" : true,
      "message" : "hi, elasticsearch"
    }

미리 아무런 매핑도 정의하지 않고 다짜고짜 위 쿼리를 실행해보았습니다. 결과는 아래와 같은데요.

    #! Deprecation: the default number of shards will change from [5] to [1] in 7.0.0; if you wish to continue using the default of [5] shards, you must manage this on the create index request or with an index template
    {
      "_index": "test",
      "_type": "default",
      "_id": "1",
      "_version": 1,
      "result": "created",
      "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
      },
      "_seq_no": 0,
      "_primary_term": 1
    }

이 상태에서 매핑 정보를 찍어보도록 하겠습니다.

    GET /test/default/_mapping

그 결과는 아래와 같습니다. 놀랍게도 날짜, bool, 텍스트 정보임을 알아채고 알아서 매핑 정보를 만들어준 것을 확인할 수 있습니다.

    {
      "test": {
        "mappings": {
          "default": {
            "properties": {
              "created": {
                "type": "date",
                "format": "yyyy/MM/dd HH:mm:ss||yyyy/MM/dd||epoch_millis"
              },
              "isExist": {
                "type": "boolean"
              },
              "message": {
                "type": "text",
                "fields": {
                  "keyword": {
                    "type": "keyword",
                    "ignore_above": 256
                  }
                }
              }
            }
          }
        }
      }
    }

text 타입을 보시면 조금 특이한 걸 보실 수 있는데요. 바로 text 타입과 keyword 타입을 동시에 가지고 있다는 점입니다. keyword 타입은 aggregation과 같이 정확한 일치가 필요한 곳에 쓰이는 타입이고, text 타입은 full text search 에 쓰이는 타입인데 둘 다 가지고 있네요? 일반적으로 text 타입은 keyword 타입을 함께 가지고 있습니다. 그리고 여기서 하나 더 주목할 점은 복수 개의 타입을 가질 수 있다는 점입니다. message 필드가 text 와 keyword 타입을 동시에 가질 수 있는 것처럼요.

# Meta fields

매핑에는 10가지의 메타 필드가 있습니다. 말 그대로 메타 정보를 담은 필드인데 유용하게 사용할 수 있는 필드들입니다. 여기서는 자주 쓰이는 몇 가지 메타 필드를 다루어보겠습니다.

1. _index : 문서가 속한 인덱스의 이름을 나타내는 필드입니다.
2. _id : 문서의 아이디를 저장하는 필드입니다.
3. _source : 처음 문서를 색인할 때 사용했던 JSON 입니다. 해당 JSON 오브젝트를 사용해서 색인을 진행하였는데, 이 오리지널 JSON은 색인되지 않았습니다. 따라서 원본 그대로 검색할 수는 없지만, 조회할 수는 있습니다.
4. _field_names : non-null 값을 가진 모든 필드의 이름을 포함하고 있습니다.
5. _routing : ES는 해시값을 이용해서 문서를 샤드에 배분하는데요. 이 과정에서 이용한 값을 저장하는 필드입니다.
6. _version : 문서가 몇 번이나 업데이트 되었는지 (= 새로 삽입되었는지) 알 수 있는 지표입니다. 생성 시 1이 부여되고 업데이트가 일어날 때마다 1이 증가합니다. 예를 들어 버전이 3인 경우 두 번의 업데이트가 진행된 것입니다.
7. _meta : ES가 간섭하지 않는 커스텀 필드입니다. 저장하길 원하는 정보를 자유롭게 저장할 수 있습니다.

# Field data types

ES를 배우면서 그렇게까지 재미있지는 않은(?) 내용인데요. 그럼에도 불구하고 확실히 알아야 하는 내용들이 분명히 있기 때문에, 중요한 내용들 위주로 살펴보도록 하겠습니다.

필드 데이터 타입은 크게 아래 네 가지로 구분할 수 있습니다.

1. Core data types
2. Complex data types
3. Geo data types
4. Specialized data types

순서대로 살펴보도록 하겠습니다.

### Core data types

이미 프로그래밍 언어를 배우면서 매우 친숙하게 접한 데이터 타입들이 여기에 있습니다. string, integer, dates, 이런 것들이죠.

1. Text : 풀텍스트 검색을 위해 주로 쓰이는 데이터 타입입니다. 제품 설명, 블로그 포스트 등이 이에 해당할 텐데요. text 데이터 타입은 대체로 원문 그대로 저장되지 않고 약간의 수정을 거친 후에 저장됩니다. 그리고 타입 특성상 sorting 이나 aggregation 에는 잘 쓰이지 않고 있습니다. 이런 것에 쓰이는 데이터 타입은 바로 keyword 입니다.
2. Keyword : 구조화된 데이터를 위해 주로 쓰입니다. 이메일이라든지, 카테고리, 태그와 같은 데이터들이요. 주로 필터링이나 aggregation을 위해 쓰입니다. text 데이터 타입과 달리 주어진 그대로 저장됩니다.
3. Numeric : 숫자 데이터들입니다. byte, short, integer, long, float, double 등이 있습니다.
4. Date : 날짜입니다. y, M, d로 적절히 나타내주기만 하면 대체로 자유롭게 사용 가능합니다.
5. Boolean : true, false를 가진 필드입니다.
6. Binary : base64로 인코딩된 string을 저장할 수 있습니다. 이미지 등에 쓰일 수 있겠지요?
7. Range : 범위를 저장할 수 있는 필드입니다. gt(greater than), gte(greater than or equal), lt(less than), lte(less than or equal) 등을 사용해서 범위를 지정할 수 있습니다.

### Complex data types

Core data type이 아니지만 평소에 접할 수 있던 타입들이 주로 여기에 속합니다. 배열이라든지, 객체와 같은 것들이요.

1. Object : JSON을 저장할 수 있습니다. nested 형태도 가능합니다. 다만 저장되는 형태가 조금 달라질 수 있는데, 예를 들면 다음과 같습니다.

        -- index time
        {
        	"name" : {
        		"firstName" : "Bk"
        		"lastName" : "Nam"
        	}
        }
        
        -- stored
        {
        	"name.firstName" : "Bk",
        	"name.lastName" : "Nam"
        }
        

2. Array : 배열 안에 들어있는 모든 데이터의 타입이 같아야 하고, nested 형태는 flatten 되어 저장됩니다. 예를 들면 아래와 같습니다.

        [ 1, [ 2, 3 ] ] => [ 1, 2, 3 ]
        ["one", "two", "three"] (O)
        ["one", 2, "three"]     (X)
        
        --
        {
            "persons" : [
                { "name" : "Bk Nam", "age": 28},
                { "name" : "HM Son", "age": 30}
            ]
        }
        
        =>
        
        {
            "persons.name" : [ "Bk Nam", "HM Son" ],
            "persons.age" : [ 28, 30 ]
        }

    이상한 점을 발견하셨나요? 우리는 JSON의 배열을 사용하지 못합니다. nested array는 ES가 모두 해체시켜버리기 때문입니다. 그러면 이러한 사항을 어떻게 해결할 수 있을까요?

4. Nested : 이 데이터 타입을 사용할 때는, 각 object가 독립적으로, 숨겨진 문서로 색인됩니다.

### Geo datay type

- Geo-point data type

말 그대로 경도, 위도를 이용해 지구에서 위치를 나타내기 위한 데이터 타입입니다. 위도(latitude)와 경도(longitude)를 짝으로 저장할 수 있으며 그 방법은 4가지가 있습니다.

    {
        "location": {
            "lat": 33.5206608,
            "lon": -86.8024900
        }
    }
    
    {
        "location": "drm3btev3e86"
    }
    
    {
        "location": "33.5206608, -86.8024900"
    }
    
    {
        "location": [-86.8024900, 33.5206608]
    }

- Geo-shape data type

필요할 경우 point, polygon, linestring, multipoint, multilinestring, multipolygon, geometrycollection, envelop, circle 등 추가로 다양한 형태의 데이터를 저장할 수 있습니다. 

### Specialized data types

IP 주소 등 특정한 데이터만 저장할 수 있도록 하는 데이터 타입입니다. 목록은 다음과 같습니다.

- IP : IPv4, IPv6 형태 모두 저장 가능합니다.
- Completion : 자동 완성 기능, 검색어 추천을 위한 데이터 타입입니다.
- Attachment : PDF, PPT, RTF와 같은 다양한 문서 포맷을 위한 데이터 타입입니다. 적용하기 위해서는 플각 문서 양식에 맞는 플러그인이 필요합니다. 텍스트 추출을 위해 내장 플러그인인 Apache Tika를 사용할 수 있습니다.

        sudo bin/elasticsearch-plugin install ingest-attachment