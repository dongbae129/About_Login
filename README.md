# About_Login
<h1>-목차-</h1>
<ul>
  <li>1.쿠키</li>
  <li>2.세션</li>
  <li>3.passport(node에서 진행)</li>
</ul>

로그인으로 바로 넘어가기 전에 http와 로그인의 관계 및 왜 쿠키, 세션이 필요한지 알아보고 넘어가겠습니다.

1.웹은 클라이언트 <-> 서버 / request <-> response 로 이루어집니다. 

2.그리고 이런 동작은 http라는 프로토콜을 이용하여 작동합니다.

3.그리고 http는 stateless라는 특징을 가지고 있는데 이는 request,response 할일 다하고 연결을 끊어버리는걸 의미합니다.

그렇다면 로그인 문제와 로그인 후 동작하는 세션들(회원정보 가져오는 것들)에 문제가 생깁니다. 이를 해결하고자 쿠키와 세션이 등장합니다.

<h3>-쿠키-</h3>
서버를 대신해서 각종 정보들을 웹 브라우저(사용자의 컴퓨터)에 저장합니다. request를 보낼때 header에 동봉하여 보내서 서버가 사용자를 식별할수 있도록 합니다.
인증 과정중 최초 request를 통해 서버가 client에게 cookie를 발급하며 다음 request부터 client가 server에게 cookie와 같이 request를 합니다.
쿠키는 key: value 형식으로 저장되며 유효기간을 설정할수도 있습니다. 유효기간이 설정되지 않았다면 웹 브라우저가 종료될때 사라지게 됩니다.
단점 : 매 request마다 header에 추가하기 때문에 트래픽 추가 발생하며 client쪽에 저장되기 때문에 보안적 문제점이 있습니다.
그렇기 때문에 쿠키는 세션관리, 개인화(광고나 장바구니 등등)를 위하여 사용됩니다.

<h3>-세션-</h3>
세션은 쿠키에서 보완을 강화한 버전입니다. 쿠키에 인증정보 같은거를 저장하지 않고 세션아이디를 저장합니다. 인증정보는 서버에 저장을 하기 때문에 이 세션아이디를 가지고 서버와
인증을 진행합니다. 서버에 정보를 저장하기 때문에 쿠키보다는 훨씬 안전합니다. 또한 외부에서 열람하더라도 인증정보와 매핑을 시킬수 없기도합니다.
즉 쿠키와 세션의 차이점은 세션은 쿠키를 이용해서 인증 정보가 아닌 세션아이디를 담아서 request를 한다.
세션은 서버메모리, DB, file에 저장할수 있다.
보안적인 이슈를 위해 httponly는 꼭 true를 설정하고 https를 적용하여 secure옵션도 true로 설정해줍시다.

<h3>-passport-</h3>
인증을 진행하는데 도움을 주는 자동화 도구입니다. passport를 사용하지 않고도 로그인 등 인증을 할수 있지만 모든 라우터에 대해 처리해야 하고 여간 귀찮은게 아닙니다. 그렇기 때문에
안전하고 편리한 passport를 적용해 줍시다.
passport는 로그인에 대해 여러가지 방법을 가지고 있습니다. 그리고 이 방법을 passport는 "전략"이라고 명칭하고 있습니다. 구글,네이버,로클 등등 매우 많습니다.
여기서 진행할 방법은 로컬 방법으로 가장 기본적인 방법이며 id,password를 db와 비교하여 진행합니다.

진행 환경은 node.js이며 mysql, sequelize를 사용합니다.
사용할 코드는 b-log 프로젝트에서 사용하는 코드의 일부를 사용합니다.

폴더구조
````
  back  
    passport  //passport폴더    
      index.js      
      local.js      
    routes  //라우터 폴더    
      user.js      
    index.js  //서버 파일
 ````
 
 먼저 passport 폴더에서
 //passport/local.js
 ````
const passport = require("passport");
const { Strategy } = require("passport-local");
const bcrypt = require("bcrypt");
const db = require("../models");

module.exports = () => {
  passport.use(
    new Strategy(
      {
        usernameField: "id",
        passwordField: "password",  (1번)
      },
      async (id, password, done) => { (2번)
        try {
          const user = await db.User.findOne({
            where: { userId: id },
            
          });
          if (!user)
            return done(null, false, { reason: "존재하지 않는 사용자 입니다" });
          const result = await bcrypt.compare(password, user.password); //password는 front에서 보낸거, user.password는 db에 있는 비번
          if (result) return done(null, user);
          return done(null, false, { reason: "비밀번호가 틀립니다" });
        } catch (e) {
          console.error(e);
          return done(e);
        }
      }
    )
  );
};
````
해당 코드는 passport에서 local에 대한 전략을 사용하여 로그인을 실행할때 passport.authenticate를 실행할때 적용되는 경우입니다.
(1번)에서 passport가 사용할 필드이름. "id"와 "password"는 client에서 보낸 이름입니다.
(2번)에서 전략입니다. 저는 mysql db를 이용하여 sequelize를 통해서 db와 비교를 하였습니다.

//passport.index.js
````
const passport = require("passport");
const db = require("../models");
const local = require("./local");
module.exports = () => {
  passport.serializeUser((user, done) => {
    return done(null, user.id); (3)
  });

  passport.deserializeUser(async (id, done) => {
    try {
      const user = await db.User.findOne({  (4)
        where: { id },
        // attributes: ["userId", "nickname"],
      });
      return done(null, user);
    } catch (e) {
      console.error(e);
      return done(e);
    }
  });
  local();
};

````

serializeUser에서 session을 확인합니다. client에서 보낸 session을 이용하여 passport가 내부적으로 해당 user를 찾아냅니다.
그리고 (3)에서 해당 user를 개인적으로 판별할 정보인 id를 done()을 통해서 반환합니다.
(4)는 (3)에서 받은 user.id를 가지고 db와 비교를 하여 있다면 user를 없다면 error를 반환합니다.
즉, serialize는 session을 통해 user.id를 가지고 오고 deserialize는 session을 통해 가지고온 user.id를 db와 비교하여 검증을 하는 시스템입니다.

여기까지가 로그인까지의 과정이며 이를 이용해서 로그인 한 유저를 검증하는 곳에 적용할수 있습니다.

````
exports.isLoggedIn = (req, res, next) => {
  if (req.isAuthenticated()) {
    next();
  } else {
    res.status(401).send("로그인을 하세요");
  }
};

exports.isNotLoggedIn = (req, res, next) => {
  if (!req.isAuthenticated()) {
    next();
  } else {
    res.status(401).send("로그인한 사용자는 접근할수 없습니다");
  }
};
````
isAuthenticated는 passport가 제공하는 함수이며 true,false를 반환합니다.

````
router.get("/", isLoggedIn, async (req, res) => {
  user정보 가져오기
});
````
이렇게 인증된 user정보를 가져와야할때 callback함수 전에 export한 isLoddedIn middleware를 추가하면 자동으로 user 인증을 할수 있습니다.

여기까지 b-log에 적용된 passport local에 적용된 사례로 알아봤으며 google,kakao로도 소셜로그인이 가능하니 passport 공식 문서에서 자세하게 참고하시면 좋을것 같습니다.
