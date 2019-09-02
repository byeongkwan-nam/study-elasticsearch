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