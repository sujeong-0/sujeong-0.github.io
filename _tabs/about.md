---
# the default layout is 'page'
icon: fas fa-info-circle
order: 4
---

<style>
.grid-container {
  display: grid;
  /*gap: 10px;  전체 그리드 간격 */
}
.line{
  display: flex;
  /* gap: 10px; 왼쪽과 오른쪽 컬럼 사이 간격 */
  /* margin-bottom: 1px; ✅ line과 line 사이 간격 추가 */
}
.grid-left {
  flex: 1;
}
.grid-right {
  flex: 3;
}
.text-left{
  text-align: left;
}
.text-right{
  text-align: left;
}
</style>

<div class="grid-container">
  <div class="line">
    <div class="grid grid-left text-right"><img width="150" src="{{ site.baseurl }} /assets/img/me.png"></div>
    <div class="grid grid-right text-grid-left">
    <br/>
    <ul>
      <li>Phone: 010-5362-6425</li>
      <li>Email: 0_sujeong@naver.com</li>
      <li>Github: <a href="https://github.com/sujeong-0">https://github.com/sujeong-0</a></li>
    </ul>
  </div>
</div>
</div>

# 자기소개

새로운 기술 트렌드를 꾸준히 공부하며, 배운 것을 공유하는 것을 즐깁니다.
사내 스터디를 주도하며 학습 문화를 형성했고, Git Flow를 도입하여 서비스 관리 체계와 협업 방식을 정리했습니다.
그 과정에서 팀원들에게 “함께 성장할 수 있는 환경을 만들어준다”는 평가를 받았으며, 문제 해결 과정에서 능동적으로 나서는 모습이 팀에 긍정적인 영향을 주었다는 피드백을 받았습니다.
앞으로는 서비스의 안정성과 확장성을 고려한 개발을 지향하며, 문제 해결과 협업의 중심에서 더 큰 역할을 할 수 있는 개발자로 성장하고자 합니다.

# 기술 스택

- Spring Boot(Java), MyBatis, JUnit, Web Socket
- MariaDB, Elasticsearch
- Linux
- Git

# 경력

### 넥타르소프트

실시간 음성 인식을 통해 119 신고 내용을 텍스트화하고, 신고자의 위치 및 상황에 맞는 대응 정보 추천 서비스



<div class="grid-container">
  <div class="line">
    <div class="grid grid-left">2020.12 ~ 2024.08<br><i style="font-size:15px">기술개발연구소</i><br><i style="font-size:15px" >웹개발팀 대리</i></div>
    <div class="grid grid-right">
      <b>AI 신고접수 시스템 웹 개발 및 유지보수 / 운영</b>
      <ul>
        <li>Elasticsearch 반경 검색 방식을 UTMK 좌표계 기반의 유클리드 거리 계산을 수행하는 script 방식에서 WGS84 좌표계 기반의 geo-distance 필터 방식으로 변경하여 거리 계산 로직을 최적화하고 성능을 개선</li>
        <li>주도적으로 유지보수성과 유연성을 확보하기 위해 Spring Boot Profile 기능을 활용해 공통 기능을 통합하고 사이트별 맞춤 기능을 분리했으며, Git Flow 전략을 도입해 사이트별 배포를 체계적으로 관리</li>
        <li>수동 테스트의 비효율성을 해결하기 위해 자발적으로 JUnit 기반 테스트 자동화를 도입하여, 테스트 시간을 케이스당 20분에서 2분으로 단축하고 유지보수성을 향상</li>
        <li>쿼리에서 불필요한 문자열 연산을 제거하고 날짜 비교를 최적화하여 인덱스 활용도를 높이고, 데이터 조회 속도를 30초에서 3초로 개선</li>
        <li>Interceptor를 활용해 사용자 인증을 강화하고, 사용자별 접근 가능한 URL을 제한하여 보안성 향상</li>
        <li>외국인 신고자를 위해 WebSocket과 번역 API를 활용한 실시간 번역 채팅 기능 개발</li>
      </ul>
    </div>  
  </div>
  <div class="line">
    <div class="grid grid-left"></div>
    <div class="grid grid-right">
      <b>레거시 코드 개선 및 java / springboot 버전 상향 (java 8→17, springboot 2.7→3.1)</b>
      <ul>
        <li>Elasticsearch 주소 검색 구조를 최적화해 검색 데이터 규모 약 45% 감소6개의 인덱스를 3개로 통합해 효율성을 향상 및 키워드당 쿼리 횟수를 4회에서 1회로 축소</li>
        <li>Spring Boot 업그레이드에 맞춰 elasticsearch-java 및 elasticsearch-rest-client로 마이그레이션하여, JSON 직렬화 최적화 및 Fluent API 적용을 통해 검색 로직의 성능과 유지보수성을 개선</li>
        <li>서비스 전반에서 서로 다르게 사용되던 핵심 키 값을 통일하고, DB 내 비일관된 데이터를 정리하여 데이터 정합성과 무결성을 확보</li>
        <li>Elasticsearch 조회 로직의 추상화를 통해 코드의 재사용성 증가</li>
      </ul>
    </div>
  </div>
  <div class="line">
    <div class="grid grid-left"></div>
    <div class="grid grid-right">
      <b>내부 백오피스 툴 개발</b>
      <ul>
        <li>정규식을 이용한 데이터 정제 자동화 기능 개발</li>
        <li>크롤링 데이터의 관리 페이지 개발크롤링 데이터의 관리 페이지 개발</li>
      </ul>
    </div>
  </div>
</div>

# 교육

<div class="grid-container">
  <div class="line">
    <div class="grid grid-left">2021.05 ~ 2024.02</div>
    <div class="grid grid-right">학점은행제 (컴퓨터 공학 학사 졸업)</div>
  </div>
  <div class="line">
    <div class="grid grid-left">2017.03 ~ 2021.02</div>
    <div class="grid grid-right">경민대학교 (융합소프트웨어과 전문 학사 졸업)</div>
  </div>
</div>

# 자격증

<div class="grid-container">
  <div class="line">
    <div class="grid grid-left">2023.11</div>
    <div class="grid grid-right">정보처리기사</div>
  </div>
</div>







