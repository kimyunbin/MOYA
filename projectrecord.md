# Project Record

> MOYA 프로젝트를 하면서 얻은점을 기록합니다.
>
> Django와 vue로 처음 개발하며 마주쳤던 여러 오류들과
>
> 이를 해결하는 과정들을 정리해둔 개발 '오답노트'입니다.

싸피에 들어와서 난생 처음 배우는 뷰와 장고들을 이용해 한학기를 마무리 하는데에 있어서 배우는 것만 사용하는 것에 그치지 않고 만들고 싶은 기능들과 기술들을 스스로 찾아보고 수 많은 에러들을 만나면서 하나하나 배워나갈 수 있었습니다. 프로젝트를 하면서 처음에는 모르면 교수님에게 도움을 요청했는데, 그렇지 못한 상황들이 더 많았고 그 상황을 해결하려는 실력 또한 많이 늘었다고 생각합니다. 또한 처음 해본 일 또한 자신감 있게 할 수 있는 기회를 얻었다고 느끼는 부분도 많았습니다. 특히나 처음 도전해본 영상 제작을 맡으면서 많은 부분을 배울 수 있는 프로젝트라 뿌듯합니다. 

## DJango

* ERD

  * 서버의 URL을 만들고 어떤 데이터를 front에 주겠다고 계획하기 위해서는 우선적으로 **우리는 어떤 사이트를 만들것인가??** 에 대해서 먼저 알아야 했다. 커뮤니티 기반의 회원제 영화 추천 사이트를 만든다는 뼈대는 기본적인 명세는 있었기에, 유저, 커뮤니티, 영화의 세가지를 가지고 유저에게 무엇을 보여주고 유저의 요청에 따라 원하는 정보를 준다는 것 자체가 우리의 모든 큰그림은 그려져 있어야 한다는 것이다. 

    이러한 것들은 처음엔 페어와 함께 상의할때 **처음 시작부터 끝을 생각한다는 것 자체가 너무 힘드니, 조금씩 만들어 보고 부족한 부분이 있다면 추가하자** 라고 처음엔 이런생각을 가졌다. 하지만 DB설계는 처음부터 잘 짜는게 나중에 추가 시키는것보다 더 노력을 적게 한다는걸 느낄 수 있었다. 수십만개가 쌓여있는 DB의 NULL을 상상하며.. 

    그러기에 어떤 기능을 가지고 있는지 페이지 개요를 짜고 그 안에 어떤 기능이 필요한지 상상하며 그에 따른 RESTFUL 한 URL까지 모두 모두 작성하고 나니 하루가 지나가버렸다. 처음엔 다른 팀을 봤을때는 바로 코딩을 하는 팀도 있어서 느려보였지만, 끝이 날땐 아니였다. `밑그림은 정말 중요하다. `

* JWT 토큰

  * form의 POST method를 사용했던 예전엔 csrf 토큰을 사용했기고, 바닐라 스크립트로 클릭시 follow 수를 불러 오고 있었다. 하지만 지금은 `JWT token`으로 회원가입과 로그인을 진행하는 상황에서 어떻게 로그인하는지 머리가 하얘졌다.

    해결방법은 데코레이터 사용으로 로그인한 사용자는 header에 jwt 토큰을 붙여서 로그인했다는것을 알리고 서버에서는 데코레이터로 올바른 로그인인지 알아냈다.

  ```django
  @api_view(['POST'])
  @authentication_classes([JSONWebTokenAuthentication])
  @permission_classes([IsAuthenticated])
  
  ```

* gravatar 설정법 

  * 유저 이메일로 그라바타 해쉬를 생성하는법

    ```python
    import urllib, hashlib # gravatar library
    email = person.email
    email_hash = hashlib.md5(request.user.email.encode('utf-8').strip().lower()).hexdigest() #gravatar hashcontext
    context ={
            'email_hash':email_hash,
        }
        return JsonResponse(context)
    ```

    * 받은 axios 요청으로 vue에서 url 링크를 넣어주고 바인딩 해줌

      d=defualt 사진 고르는 방법 

    ```javascript
    this.imgUrl =`https://s.gravatar.com/avatar/${res.data.email_hash}?d=identicon`
    ```

  * vue-gravatar 방법

    * ```vue
      <v-gravatar email="somebody@somewhere.com" />
      ```

    * 사용법이 간단해서 이후 서버에서 별도의 해쉬값을 넘겨주지 않고 front에서 해결함 

* community-create

  - user를 알 수 있는 방법은 jwt 토큰의 데코레이터로 알 수있다!!
  - 커뮤니티의 리뷰를 만드는데 영화의 id만 필요한게 아니라 역참조로 특정 영화의 정보까지 모두 알고 싶었음
  - 리뷰 시리얼라이저 안에 movie = MovieSerializer(read_only=True,many=True)로 줬는데 참조가 안됨
    - 특정 무비 하나만 타겟이 되기때문에 many=True를 빼줘야함.
    - n:m 관계와 외부키들을 현재 사용하지 않기 때문에, read_only_fields로 두었음

* Serializer

  * review의 모델링 자체가 user와 movie를 외부키로 참조, 그리고 3가지 m:n으로 꼬여져 있어서 복잡함

  * 리뷰페이지에 필요한 user와 movie는 역참조로 정보를 모두 들고오고, 그 외 리뷰안의 comment와 댓글 수까지 모든것을 담아서 보내야한다는 것. 

  * ```python
    class ReviewSerializer(serializers.ModelSerializer):
        user = UserDetailSerializer(read_only=True)
        movie = MovieDetailSerializer(read_only=True)
        comment_set = CommentSerializer(read_only=True,many=True)
        comment_count = serializers.IntegerField(source='comment_set.count', read_only=True)
    
        
        class Meta : 
            model = Review
            fields = "__all__"
            # exclude = ('like_users','funny_users','helpful_users')
            read_only_fields =('movie','user','like_users','funny_users','helpful_users')
    ```

  * 알수 없는 시리얼라이저 에러 get-object_or_404로 단일 review를 들고왔음에도 리스트로 묶어주고 그 데이터를 many=True로 해줘야하는 이상한 상황 ... 잘모르겠지만 이렇게 해야만 사용 가능하다. 

    ```python
    review = get_object_or_404(Review, id=review_id)
    review_list = [review]
    
    reviewserializer = ReviewSerializer(data=review_list, many=True)
    ```
  
    

## Vue

- 회원가입후 로그인 정보를 건네주는가의 고민

  - 회원가입시 axios 요청을 보내는데, 필요한 username,password, email 을 v-model로 묶어서 요청을 보냄.
  - 회원가입 응답을 정상적으로 받았을때 구글로그인처럼 바로 로그인 할 수 있게 할것인지, 다시 유저가 쓰게 만들것인지 고민함.
    - 해킹의 위험이 있으므로 응답받은 후 username과 password 데이터 초기화로 다시 유저가 쓰게함.
    - 회원가입이 잘 되었으면 토글을 작동시켜 login 페이지로 전환

- 로그인시 broken pipe 문제

  - 에러 이슈 초반, 로그인이 잘될때도 안될때도 있는데, 에러시 broken pipe 라고 서버 응답이 뜸

  - 로그인 클릭시 로그인 form의 기본 기능인 get 요청 실행 인지

    - preventdefault()로 막아도 오류가 계속 나옴

    - 해결방법은 html에 submit 속성  막아버린 후  정상 작동

      ```vue
      <form v-on:submit.prevent="login">
      ```

- 회원가입 버튼을 눌리면 (default 값) 로그인 페이지를 보여주는 문제

  ​	-  현재 로그인/회원가입 페이지의 동적 디자인을 위해 같은 html 파일 안에서 토글로 작동하는 상황임 

  - 페이지 자체가 토글을 누르면 회원가입 폼을 보여주는 형태로
  - app.vue에서 signup 버튼을 누르면 props로 true값을 넘겨준다음
  - login.vue에서 created 시 true 값이라면 toggle 실행하여 page 전환 시도
  - 자꾸 false로 값이 변경되지않아 보류함 

- jwt 토큰만으로 마이페이지 들어가기

  유저정보 동적라우팅 관련해서 어떻게 하는지 모르겟음

  ```jsx
  {
      path:'/mypage/:username',
      name:'Mypage',
      component: Mypage,
    }
  ```

  - router 단에서 동적 라우팅하는법 `:username` 의 의미가 동적라우팅 자리

  - app.vue에서 자신의 mypage를 들어가는법

    - 자신을 알려면 알고 있는건 jwt밖에 모르는데 역으로 username이 필요하다...
    - https://www.npmjs.com/package/jwt-decode
    - 이럴떈 `jwt-decode` 를 이용하여 디코딩해서 로그인한 유저의 username을 얻을 수 있다.
    - 라우터링크와 라우터의 링크를 맞추는 법 `<router-link :to="/mypage/${username}"`  디코딩 해서 가져온 Username으로 마이페이지를 찾을 수 있다.

    ```jsx
    const decoded = jwt_decode(this.token)
        // console.log(decoded);
        this.username = decoded.username
    ```

- [해결]하트 위치 내맘대로 안움직임

![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/bbc59d33-8e95-48c3-a09b-e160a3ed28f8/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210603%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210603T082310Z&X-Amz-Expires=86400&X-Amz-Signature=3a5857b6253d2f2871c02eb283f9ccdc859acdbd24825dd0c4512ded6103342a&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22)

- transform: translate(130%,-190%);  css 재배치로 해결

- 이름 옆에 하트를 두려고 했는데 UI적으로 어디가 제일 이쁠까 고민끝에 follow 버튼 사이에 위치 

- 기존의 레퍼런스 툴이 바닐라 js로 이루어져 있어서 매우 헷갈림

  ```jsx
  if (!res.data.follow){
      content.setAttribute('class','content heart-active')
      heart.setAttribute('class','heart heart-active')
  } else{
    content.setAttribute('class','content')
    heart.setAttribute('class','heart')
  
  }
  ```

  setttribute로 서버에서 넘어온 follow 타입을 기준으로 토글형식 완성해줌

- userfollow component에서 axios 요청을 보냇기에 유저 팔로우 수는 props/emit으로 값을 넘겨준 다음 innerText로 변경

  ```jsx
  countFollow: function (data) {
        // console.log(data.count);
        const followCount =document.querySelector('#following')
        followCount.innerText =data
      }
  ```

- 자기 자신을 팔로우 하지 못하게 해야하는 문제

  - jwt decode로 request.user 와 현재 페이지 유저 비교후 v-if="isUser"로 처리

  ```jsx
  created: function (){
      const decoded = jwt_decode(this.$store.state.token)
      // console.log(decoded);
      const pagename = this.$route.params.username
      const username = decoded.username
      // console.log(pagename);
      // console.log(username);
      if (pagename === username){
        this.isUser = false
      } else {
        this.isUser =true
      }
    }
  ```

- review 전체리스트 반응형 grid 만들기

  - reviewList에서 axios 요청으로 리뷰전체중 20개정도 들고 어레이를 받아서 reviewListitem으로 하위 컴포넌트 for문을 돌림
  - 리뷰카드가 한줄에 3개씩, 반응형 그리드로 만들고 싶었는데 자꾸 오류가 남.
    - row-다음-하위 컴포넌트에서 col을 나눠 줬는데 그위에 있는 body 태그로 인해 모든 컴포넌트들이 col을 가지지 못함.
    - 그 안의 body 태그의 css 속성이 vw,vh 속성을 가지고 있어서 제대로 작동이 안됨
      - 상위 컴포넌트에서 class="col-sm-12 col-md-6 col-lg-4"을 줘서 1,2,3개 나오게 만듬
      - body 태그 지워버린 다음 css 조정

- 상대시간 만들기 

  - 리뷰디테일, 리뷰전체페이지 등 여러곳에서 사용하기 위해 mixin의 filters안에 작성

  - 작성시간을 매개변수로 가져와서 페이지를 바라보는 시간을 today로 상대시간을 설정함 

  - 방금전, xx분전, xx시간전, xx일전, x년전으로 상대시간을 표현함

  - ```js
    timeFor : function(created_at){
    			// console.log(created_at)
    			const today = new Date();
    			const timeValue = new Date(created_at);
    
    			const betweenTime = Math.floor((today.getTime() - timeValue.getTime()) / 1000 / 60);
    			if (betweenTime < 1) return '방금전';
    			if (betweenTime < 60) {
    					return `${betweenTime}분전`;
    			}
    
    			const betweenTimeHour = Math.floor(betweenTime / 60);
    			if (betweenTimeHour < 24) {
    					return `${betweenTimeHour}시간전`;
    			}
    
    			const betweenTimeDay = Math.floor(betweenTime / 60 / 24);
    			if (betweenTimeDay < 365) {
    					return `${betweenTimeDay}일전`;
    			}
    
    			return `${Math.floor(betweenTimeDay / 365)}년전`;
    		}
    ```

  - mixin을 import 시켜준다음 사용하는법은 `{{comment.created_at | timeFor}}` 로 사용할 수 있다. 

- 댓글 작성시 새로고침 없이 댓글 새로고침 만들기 

  - comment axios를 정상적으로 응답 받았다면, 현재 페이지를 요청을 다시 보내는 method를 다시 실행`this.getData() ` 을 불러와서 페이지에 업데이트된 부분을 다시 불러오는 방법을 사용함 

- 비로그인시 로그인페이지로 이후 원래 가려는 페이지로 만들기

  - 초기에 next url을 사용하는 부분이 영화월드컵으로 동적 라우팅을 사용하지 않는 페이지에 적용함

  - 이후에 페이지의 보안을 위해 리뷰디테일, 무비디테일, 마이페이지 등등 비로그인시 보여지는 부분과 로그인시 보여주는 부분을 나누고 비로그인시 접근하려면 로그인페이지로 보내고 그다음 next URL을 저장해서 로그인시 보내주는 login redirect를 vue에서 만드려고 함

  - 동적라우팅을 사용한 페이지가 있기에 router name으로만으로는 만들기 힘들었고, page name 과 동적라우팅 변수를 모두 vuex에 저장시킴

    ```js
    /* 토큰이 있는지 확인하고 없으면 페이지 name과 동적라우팅 변수를 params로 저장후 detailItem으로 딕셔너리로 vuex의 store에 저장시킴 */
    loginCheck : function(){
          if (!this.$store.state.token){
            const detailItem ={
              name: 'community',
              params: this.$route.params.detail
            }
            this.$store.dispatch('setNextPage',detailItem)
            this.$router.push({name : 'Login'})
          }
    ```

  - 로그인을 axios 요청을 응답받은 후에 nextpage 와 nextparams가 있는지 먼저 확인후 있다면 다음 페이지로 push후 vuex 초기화, 아니라면 메인페이지로 이동함

    동적 라우팅때문에 push 변수도 ``${this.nextPage}/${this.nextparams}`` 으로 명시하여 보내주는편이 편함.

    ```js
    .then(res => {
              console.log(res)
              this.$store.dispatch('login',res)
              // nextPage가 있다면 보내주기
              if (this.nextPage){
                if (this.nextparams){
                  this.$router.push(`${this.nextPage}/${this.nextparams}`)
                  this.setNextPage('')
                }else {
                  this.$router.push({ name: this.nextPage ,})
                  this.setNextPage('')
    
                }
              }else{
                this.$router.push({ name: 'Movies'})
              }
    ```

    

## CSS

> 사용자에게 깔끔하고 이쁜 그리고 동적으로 구성함으로써 재밌는 페이지를 만들기 위해 노력했습니다. 
>
> 그 중 로그인 페이지의 전환 방법이나, 하트의 애니메이션 구성, hovering 된 페이지 등등 만들면서 공부한 css를 기록했습니다. 



* 단위

  * px : 가장 기본적인단위, 픽셀단위라는 고정값에 따라 정해짐 
  * em:부모 요소를 기준으로 자식 요소의 크기를 정하는 것
  * rem(root em) :  부모 요소를 기준으로 자식 요소의 크기를 정하는것을 가장 최상단(root) 기준으로 맞추는 것 . 최상위 태그 html에 정의된 사이즈를 기준으로 배수하겠다는 것을 의미
  * vh(vertical height): 화면 크기의 높이에 비례. 1vh = 실제 높이값의 1/100
  * vw(vertical width): 화면 크기의 너비에 비례. 1vw = 실제 너비값의 1/100 
  * vmin / vmax : 너비값과 높이값에 따라 최대, 최소값을 지정할 수 있음 

* margin 

  * 요소의 네 방향 바깥 여백 영역 설정

* padding

  * 요소의 네 방향 안쪽 여백 영역 설정 

* box-sizing: border-box;

  *  요소의 너비와 높이를 계산하는 방법을 지정하는데, 테두리와 안쪽 여백의 크기도 요소의 크기로 고려함. 안그럼 이상해짐

* position

  * 문서 상에 요소를 배치하는 방법, static, relative, absolute, fixed, sticky 가 있다. 
  * static : 디폴트 값(기준 위치)
    * 기본적인 요소의 배치 순서에 따름(좌측 상단)
    * 부모 요소 내에서 배치될 때는 부모 요소의 위치를 기준으로 배치
  * relative
    * static 위치를 기준으로 이동(상대 위치)
  * absolute
    * static이 아닌 가장 가까이 있는 부모/조상 요소를 기준으로 이동( 절대 위치)
  * fixed
    * 부모 요소와 관계 없이 브라우저를 기준으로 이동(고정 위치)
    * 스크롤시에도 항상 같은 곳에 위치 

* display

  * `display :block`
    * 줄 바꿈이 일어나는 요소
    * 화면 크기 전체의 가로 폭을 차지한다.
    * 블록 레벨 요소 안에 인라인 레벨 요소가 들어갈 수 있다.
    * 대표적인 인라인 레벨요소
      * div / ul, ol, li / p / hr / form 등
  * `display: inline`
    * 줄 바꿈이 일어나지 않는 행의 일부 요소
    * content 너비만큼 가로 폭을 차지한다.
    * width, height, margin-top, margin-bottom을 **지정할 수 없다**.
    * 상하 여백은 line-height로 지정한다
    * 대표적인 인라인 레벨 요소
      * span / a / img / input, label / b, em, i , strong 등
  * `display: inline-block`
    * block과 inline 레벨 요소의 특징을 모두 갖는다.
    * inline 처럼 한 줄에 표시 가능하며, 
    * blcok처럼 width, height, margin 속성을 모두 지정할 수 있다.
  * `display: none`
    * 해당 요소를 화면에 표시하지 않는다.( 공간조차 사라진다.)
    * 이와 비슷한  visibility: hidden은 해당 요소가 공간은 차지하나 화면에 표시만 하지 않는다.

* transition

  *  CSS 속성을 변경할 때 애니메이션 속도를 조절하는 방법을 제공. 속성 변경이 즉시 영향을 미치게 하는 대신, 그 속성의 변화가 일정 기간에 걸쳐 일어나도록 할 수 있음
  * **transition-duration** : transition-duration은 변화가 몇초, 또는 몇 밀리세컨드(1/1000초)에 걸쳐 일어날지를 설정한다. 6s, 0.5s, .3s, 120ms 이라고 적혀 있다면, 이는 각각 6초, 0.5초, 0.3초, 120밀리세컨드 라는 뜻

* object-fit 

  * img나 video 같은 요소에 콘텐츠 크기를 어떤 방식으로 조절해 요소에 맞출지 결정
  * fill, contain, cover, none, scale-down 의 방식이 있음

* pointer-events 

  * `pointer-event` 속성을 통해 엘리먼트가 마우스 이벤트(호버, 클릭, 드래그 등)에 어떻게 반응할지를 지정할 수 있다. 대부분의 속성 값은 SVG 전용이므로, `pointer-events: none`을 설정하여 마우스 이벤트의 타겟이 될 수 없게 함 

* @media

  * 반응형 웹에 대한 미디어쿼리 사용

  * ```
    /* 데스크탑에서 사용될 스타일을 먼저 작성합니다. */
    
    @media screen and (max-width: 768px) {
        /* 모바일에 사용될 스트일 시트를 여기에 작성합니다. */
    }
    
    
    출처: https://offbyone.tistory.com/121 [쉬고 싶은 개발자]
    ```

  * ```
    /* 모바일에 적용될 스타일을 먼저 작성합니다. */
    
    @media screen and (min-width: 769px) {
       /* 데스크탑에서 사용될 스타일을 여기에 작성합니다. */
    }
    
    
    출처: https://offbyone.tistory.com/121 [쉬고 싶은 개발자]
    ```

* transform: translate()

  * translate (x, y) 함수는 요소를 왼쪽에서부터 x거리  위에서부터 x 거리만큼 상대적으로 위치를 정하거나, 이동 및 재배치를 지정합니다.

css 너무많아서 계속 추가...

